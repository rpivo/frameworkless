## Adding a Build Script

#### Last Updated: May 16, 2021

The followinup build script uses Rollup to bundle source code based on an entry file **src/index.js**. The `-c` flag tells Rollup to look for a config file to use (**rollup.config.js**).

```json
"build": "rollup -c"
```

Below is a minimal Rollup config with the bare essentials. This config specifies an entry point (`input`), an output directory `output.dir`, and an output format (`output.format`). This will output the built code as ES modules.

```js
export default {
  input: "src/index.js",
  output: {
    dir: "dist",
    format: "esm",
  },
};
```

### Resources

- [Rollup](https://rollupjs.org/guide/en/)
