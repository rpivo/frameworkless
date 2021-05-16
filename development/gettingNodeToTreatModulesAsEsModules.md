## Getting Node to Tread Modules as ES Modules

#### Last Updated: May 16, 2021

In order to use the familiar ES module `import` and `export` syntax in Node during development, you can set the `type` key in **package.json** to `module`. This allows you to use this same syntax in Node as well as in the browser. _You will need the latest LTS version of Node to do this._

```json
"type": "module"
```

This will allow you to prepare your modules as ES modules:

```js
import fs from 'fs';

export default { ... }
```

### Resources

- [Node / Modules: Packages](https://nodejs.org/api/packages.html)
