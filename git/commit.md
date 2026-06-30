
- [1. Structure](#1-structure)
- [2. How to commit](#2-how-to-commit)


# 1. Structure

Commit message must follow this [structure](https://www.conventionalcommits.org/en/v1.0.0/):

```txt
<type>(<optional scope>): <description>

[optional body]

[optional footer(s)]
```

- **type**: The structural intent of the code change (e.g., feat, fix).
  - **feat** (minor): A new user-facing feature or capability for the application
  - **fix** (patch)
  - **docs**
  - **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons)
  - **refactor**: A code change that neither fixes a bug nor adds a feature (restructuring logic)
  - **perf**: A code change that improves overall runtime performance
  - **test**
  - **build**: Changes affecting build systems, configuration setups, or external dependencies
  - **ci**: Changes to continuous integration configuration files and pipeline scripts (e.g., GitHub Actions)
  - **chore**: Other changes that don't modify source or test files (e.g., updating .gitignore)
  - **revert**
- **scope**: (Optional) A noun providing localized context about where the change happened, wrapped in parentheses. (e.g., component: auth, api, deps; branch: main, release; file name).
- **description**: A short summary of the code change. use present tense, Do not capitalize first letter, Do not put a period at the end.
- **body**: (Optional) explain the why and what of the change. It must begin after a single blank line following the header.
- **footer**: (Optional) Used for tracking metadata, such as external issue trackers (Fixes #12) or explicitly announcing breaking changes. It must begin after a single blank line following the body or header.

**Example: A standard patch fix with an issue link**

```txt
fix(ui): correct profile image avatar sizing on mobile screens

The user avatar container was overflowing on viewports smaller than 360px
due to a fixed flex-basis width. Swapped to percentage constraints.

Fixes: #482
```

**More Conventions**

- Use the [50-72-rule](https://preslav.me/2015/02/21/what-s-with-the-50-72-rule/) to keep your commit message short and readable. Body may consist of any number of newline separated paragraphs.
- [Described here](https://www.conventionalcommits.org/en/v1.0.0/): footers other than `BREAKING CHANGE: <description>` may be provided and follow a convention similar to [git trailer format](https://git-scm.com/docs/git-interpret-trailers) (`<key>=<value> or <key>:<value>`). When reading trailers, there can be no whitespace before or inside the `<key>`. There can be whitespaces before, inside or after the `<value>`. The `<value>` may be split over multiple lines with each subsequent line starting with at least one whitespace:

  ```
  key: This is a very long value, with spaces and
    newlines in it.
  ```

  multiple footers:

  ```
  Signed-off-by: Alice <alice@example.com>
  Signed-off-by: Bob <bob@example.com>
  Fixes: #12
  Closes: #13
  Reviewed-by: Jane Doe
  Co-authored-by: John Smith <john@example.com>
  ```

  Note that `key` use `-` in place of whitespace characters, e.g., `Acked-by`

**Declaring a breaking change**

- Place an `!` immediately after the **type** or **scope** in the header.
- In the footer start with `BREAKING CHANGE:` followed by a description

```txt
refactor(api): drop support for v1 endpoints

The v1 REST API paths are completely deprecated and removed from the active router.

BREAKING CHANGE: Any API integrations pulling data from `/api/v1` must migrate to `/api/v2`.
```

**Example: Very long descriptions**

```txt
refactor(auth)!: migrate session storage to Redis Cluster

The previous memory-cache implementation caused state desynchronization
across our multi-tenant horizontal container scaling pods. This update
shifts all active user session verification routines to use a centralized
Redis cluster backend, resolving intermittent authentication dropouts.

1. Added automatic reconnection retry backoff algorithms.
2. Implemented encrypted token serialization schemas.

BREAKING CHANGE: The local environment configurations now require a valid
`REDIS_URL` environment variable. If this connection string is omitted,
the application server will fail to initialize and crash on startup.
Additionally, the legacy `JWT_LOCAL_SECRET` variable is fully deprecated
and no longer evaluated by the authentication service runtime layer.
Fixes #10
```

- The Header Line well under the 50-character ceiling
- The Long Body Text: Instead of letting the text stretch infinitely across the screen, every single line breaks cleanly at or before the 72-character mark.
- The Breaking Change Footer: multi-line migration instructions are wrapped cleanly within the 72-character boundary line limit

# 2. How to commit

set vscode or whatever as default editor to input your commit message
```bash
git config --global core.editor "code --wait"
```

commit your message
```bash
git commit
```