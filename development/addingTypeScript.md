## Adding TypeScript

#### Last Updated: May 19, 2021

To add TypeScript for a Rollup-built repository, you can add these packages as dev dependencies:

```sh
npm i -D typescript @rollup/plugin-typescript
```

To initialize the tsconfig:

```sh
tsc --init
```

You will likely need to change a few properties in the tsconfig to interoperate with Rollup. The values below are not required, but may be good starting points.

```js
    "target": "ESNEXT",
    "module": "ESNEXT",
    "outDir": "./dist",
    "rootDir": "./src",
```

It may also be good to uncomment `Strict Type-Checking Options` and `Additional Checks` to improve type checking.

Inside **rollup.config.js**, you will need to import the TypeScript Rollup plugin to be used inside the plugins array.

```js
import typescript from "@rollup/plugin-typescript";

export default {
  input: "src/index.ts",
  output: {
    dir: "dist",
    format: "esm",
  },
  plugins: [typescript()],
};
```
