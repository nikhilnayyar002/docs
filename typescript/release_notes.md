

# TypeScript 4.9

## `satisfies`

```ts
type Colors = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];
const palette: Record<Colors, string | RGB> = {
    red: [255, 0, 0],
    green: "#00ff00",
    bleu: [0, 0, 255]
};

// property 'toUpperCase' does not exist on type 'string | RGB'.
const greenNormalized = palette.green.toUpperCase();
```
`satisfies` tells TypeScript "infer the type based on the code you see, and give me an error if the resulting type is incompatible with Person".
```ts
const palette = { ... } satisfies Record<Colors, string | RGB>;
// toUpperCase() method is still accessible!
const greenNormalized = palette.green.toUpperCase();
```
More details - https://stackoverflow.com/questions/78636946/what-are-the-differences-between-typescript-s-satisfies-operator-and-type-assert

