https://mswjs.io/docs/

- [1. ?](#1-)
  - [1.1. Quick start](#11-quick-start)
  - [1.2. Default behaviors](#12-default-behaviors)
  - [1.3. Limitations](#13-limitations)
- [2. Integrations](#2-integrations)
  - [2.1. Browser integration](#21-browser-integration)
  - [2.2. Node.js integration](#22-nodejs-integration)
- [3. Mocking HTTP](#3-mocking-http)
  - [3.1. Intercepting requests](#31-intercepting-requests)
    - [3.1.1. ?](#311-)
    - [3.1.2. Path parameters](#312-path-parameters)
    - [3.1.3. Query parameters](#313-query-parameters)
    - [3.1.4. Request body](#314-request-body)
    - [3.1.5. Request cookies](#315-request-cookies)
  - [3.2. Handling requests](#32-handling-requests)
  - [3.3. Mocking responses](#33-mocking-responses)
    - [3.3.1. ?](#331-)
    - [3.3.2. Network errors](#332-network-errors)
    - [3.3.3. Binary responses](#333-binary-responses)
    - [3.3.4. Cookies](#334-cookies)
    - [3.3.5. Redirects](#335-redirects)
    - [3.3.6. Polling](#336-polling)
    - [3.3.7. Streaming](#337-streaming)
    - [3.3.8. Response timing](#338-response-timing)
    - [3.3.9. File uploads](#339-file-uploads)
    - [3.3.10. Proxying requests](#3310-proxying-requests)
    - [3.3.11. Response patching](#3311-response-patching)
- [4. Best practices](#4-best-practices)
- [5. Recipes](#5-recipes)
  - [5.1. Higher-order resolver](#51-higher-order-resolver)


# 1. ?

## 1.1. Quick start

```
npm i msw --save-dev
```

## 1.2. Default behaviors

Every intercepted request “falls through” the list of your handlers
```js
import { http, HttpResponse } from 'msw'
 
export const handlers = [
  http.get('/user', () => console.log('One')), // only intercept, prints "One"
  http.get('/user', () => HttpResponse.json({ name: 'John' })), // request handled
  http.get('/user', () => console.log('Three')), // never reached
]

const worker = setupWorker(...handlers)
worker.listen()
```
```ts
// "request handlers" can be added after intiating the server/worker.
// the below route will override /user handlers passed above
server.use(
  http.get('/user', () => HttpResponse.json({ name: 'overrided' })),
)
```

## 1.3. [Limitations](https://mswjs.io/docs/limitations)

# 2. Integrations

## 2.1. Browser integration

create `mockServiceWorker.js` in public directory. Also [see](https://mswjs.io/docs/best-practices/managing-the-worker#updating-the-worker-script)
```bash
npx msw init public --save
```
navigate to the `/mockServiceWorker.js` URL of your application in your browser. You should see the worker script contents.

```ts
// mocks/handlers.ts
import { http, HttpResponse } from 'msw'
export const handlers = [
  http.get('https://api.example.com/user', () => {
    return HttpResponse.json({ ... });
  }),
]

// mocks/browser.ts
import { setupWorker } from 'msw/browser'
import { handlers } from './handlers'
 
export const worker = setupWorker(...handlers)

// main.ts
// registering the Service Worker is an asynchronous operation
async function enableMocking() {
  if (process.env.NODE_ENV !== 'development') return;
  const { worker } = await import('./mocks/browser')
  // returns a Promise that resolves once the Service Worker is up
  return worker.start()
}
enableMocking().then(() => {
  ReactDOM.render(<App />, rootElement)
})
```
open the Console in the Developer Tools
```
[MSW] Mocking enabled.
```

## 2.2. Node.js integration

MSW enables API mocking by patching native request-issuing modules, like `http` and `https`

```ts
import { setupServer } from 'msw/node'
import { handlers } from './handlers'
 
export const server = setupServer(...handlers)

// src/index.js
import { server } from './mocks/node'
server.listen()
// https://mswjs.io/docs/api/life-cycle-events
server.events.on('request:start', ({ request }) => {
  console.log('MSW intercepted:', request.method, request.url)
})
// ...the rest of your Node.js application.
```

# 3. Mocking HTTP

## 3.1. Intercepting requests

### 3.1.1. ?

```ts
//       👇 This is a predicate.
http.get('/resource', () => {})
//                    👆 This is a resolver.
```

You can provide a string as a request handler predicate. MSW will use [path-to-regexp@6](https://github.com/pillarjs/path-to-regexp/tree/6.x) to match your predicate

```ts
// Relative URL
http.get('/users/:id', () => {})
// Absolute URL
http.post('https://api.github.com/repos/:owner/:repo', () => {})
```
**Special tokens**
- `*` (wildcard), to match any URL or a portion of the URL in its place;
- `:foo` (parameter), to match a named parameter in the URL.

[**Regular expression**](https://mswjs.io/docs/http/intercepting-requests/#regular-expression)

You can provide a regular expression as a request handler predicate ...

**Custom predicate function**

function must return a boolean indicating whether the intercepted request should match your request handler
```ts
http.get(
    ({ request, cookies }) => (new URL(request.url)).searchParams.has('mock'),
    resolver)
```

[**Response resolver**](https://mswjs.io/docs/http/intercepting-requests/#response-resolver)
```ts
http.get('/resource', ({ request, params, cookies }) => {})
```

### 3.1.2. Path parameters

```ts
http.get<{ id: string }>('/posts/:id', ({ params }) => {
  const { id } = params
})
// ✅ /posts, /posts/abc-123 ❌ /posts/abc-123/edit
http.get<{ id?: string }>('/posts/:id?', ... )

// /settings/user : ["user"], /settings/user/privacy :["user", "privacy"]
http.get<{ segments: string[] }>('/settings/:segments+', ...)
```

### 3.1.3. Query parameters

```ts
http.get('/product', ({ request }) => {
  const url = new URL(request.url)
  const productId = url.searchParams.get('id')
  if (!productId) return new HttpResponse(null, { status: 404 })
  return HttpResponse.json({ id: productId })
})

// Given a request url of "/products?id=1&id=2&id=3", productIds = ["1", "2", "3"]
const productIds = url.searchParams.getAll('id')

const url = new URL(request.url)
url.searchParams.set('id', 'another-id')
// Create a proxy request that extends the intercepted request
const proxyRequest = new Request(url, request)
```

### 3.1.4. Request body

ou can read the intercepted request’s body as you normally would any Fetch API [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request).
```ts
http.post('/posts', async ({ request }) => {
  const newPost = await request.clone().json()
})
```
It’s highly recommended to clone the intercepted request before reading its body. While it’s okay to read the request body directly (`await request.json()`), reading it without cloning in passthrough/bypass scenarios will result in an exception (streams cannot be read twice).

### 3.1.5. Request cookies
```ts
http.get('/api/user', ({ cookies }) => {
  if (!cookies.authToken) return new HttpResponse(null, { status: 403 })
  return HttpResponse.json({ name: 'John' })
})
```

## 3.2. Handling requests

**Respond with a mocked response**

return a `Response` instance from the response resolver

**Passthrough**
```ts
import { passthrough } from 'msw'
http.get('https://api.example.com/resource', async ({ request }) => {
  const data = await request.clone().json()
  if (data?.id === 'abc-123') return HttpResponse.json({ id: 'abc-123' })
  // Let it go out to the real internet.
  return passthrough()
})
```

**Return nothing**
```ts
export const handlers = [
  http.get('/resource', ({ request }) => {
    console.log('Request intercepted!', request.method, request.url)
  }),
  http.get('/resource', () =>  HttpResponse.text('hello world')),
]
```

## 3.3. Mocking responses

### 3.3.1. ?

You can return a plain Fetch API [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) instance, too. It’s recommended to use the `HttpResponse` class instead, which is a drop-in replacement for `Response` but provides improved developer experience and unlocks otherwise unavailable features, like mocking the `set-cookie` header.

> [!note]
> `HttpResponse.text` etc is static method, `new HttpResponse(...)` is instance

```ts
  http.get('/resource', () => HttpResponse.text('hello world'))
  http.get('/resource', () => new HttpResponse(null, { status: 404 }))
```
If you throw a `Response` instance at any point in the response resolver, that response will be used as the mocked response for the request.
```ts
function withAuthorization(request: Request) {
  if (!request.headers.get('authorization'))
    throw HttpResponse.text('Unauthorized', { status: 401, headers: { ... } })
}
 
http.get<never, never, { id: string }>('/resource', ({ request }) => {
  withAuthorization(request)
  return HttpResponse.json({ id: 'abc-123' })
})
```

### 3.3.2. Network errors

results in a network error, aborting the pending request
```ts
http.get('/resource', () => HttpResponse.error())
```
client will receive a generic `TypeError: Failed to fetch` error that you should handle in the `.catch()` closure of your request.

### 3.3.3. [Binary responses](https://mswjs.io/docs/http/mocking-responses/binary)

respond with binary data, like images, video, or audio. ...

### 3.3.4. [Cookies](https://mswjs.io/docs/http/mocking-responses/cookies)

### 3.3.5. Redirects

```ts
http.get('/a', () => 
    new HttpResponse(null, { status: 301, headers: { location: '/b' }}))
```

### 3.3.6. [Polling](https://mswjs.io/docs/http/mocking-responses/polling)

Yield different responses on subsequent requests. ...

### 3.3.7. [Streaming](https://mswjs.io/docs/http/mocking-responses/streaming)

### 3.3.8. Response timing

use [delay](https://mswjs.io/docs/api/delay).

`function delay(duration?: number): Promise<void> {}`, 

`function delay(mode?: 'real' | 'infinite'): Promise<void> {}`

```ts
http.get('/resource', async () => {
  await delay(500)
  return HttpResponse.json({ id: 'abc-123' })
})
```

**Global delay**
- Option 1: Delay fallthrough handler
  ```ts
  export const handlers = [
    http.all('*', async () => { await delay(500) }),
    // ...other request handlers.
  ]
  ```
- Option 2: Higher-order resolver
  ```ts
  import { delay, type HttpResponseResolver } from 'msw'
  export async function withDelay(resolver: HttpResponseResolver) {
    return async (...args) => {
      await delay() // random realistic response time
      return resolver(...args)
    }
  }

  http.get('/user',
    withDelay(({ request }) => HttpResponse.json({ id: 'abc-123' }))
  ),
  ```

### 3.3.9. [File uploads](https://mswjs.io/docs/http/mocking-responses/file-uploads)

### 3.3.10. [Proxying requests](https://mswjs.io/docs/http/mocking-responses/proxying-requests)

### 3.3.11. [Response patching](https://mswjs.io/docs/http/mocking-responses/response-patching)

# 4. [Best practices](https://mswjs.io/docs/best-practices/)

# 5. [Recipes](https://mswjs.io/docs/recipes/custom-worker-script-location)

## 5.1. Higher-order resolver
```ts
// mocks/middleware.js
export function withAuth(resolver) {
  return (input) => {
    const { request } = input
    if (!request.headers.get('Authorization'))
      return new HttpResponse(null, { status: 401 })
    return resolver(input)
  }
}

  http.post('/comment', withAuth(({ request }) => ... ))
```
