## Minifying Source Code During the Build Process

#### Last Updated: May 16, 2021

You can use the **rollup-plugin-terser** NPM package to minify source code with Rollup. To do this, import terser in your **rollup.config.js**, and use it as an output plugin.

```js
import { terser } from "rollup-plugin-terser";

export default {
  input: "src/index.js",
  output: {
    dir: "dist",
    format: "esm",
  },
  plugins: [terser()],
};
```

### Resources

- [NPM / rollup-plugin-terser](https://www.npmjs.com/package/rollup-plugin-terser)
- [Rollup](https://rollupjs.org/guide/en/)
- [Rollup / Using Output Plugins](https://rollupjs.org/guide/en/#using-output-plugins)
