## Adding a Clean Script

#### Last Updated: May 19, 2021

This script:

- Removes the output folder for a project (this could be something like `dist` or `build` or `lib`).
- Remakes the folder.
- Copies the entry file (`index.html`).
- Copies each asset from a given assets folder.

It can be extended to copy CSS or other files that may not need to be processed by a build tool like Rollup or Webpack.

<hr />

First, add the build script to **package.json**. This runs the **clean.js** process with Node. The module can be kept in a **util/** directory at the root of the project.

```json
"clean": "node ./util/clean.js"
```

The following code will go inside **clean.js**.

```js
import fs from "fs";
import path from "path";

const cwd = process.cwd();

const assetsFolder = "assets";
const distFolder = "dist";
const srcFolder = "src";
const entryFile = "index.html";

/**
 * copies each asset from a given assets folder to the output folder.
 */
function copyAssets() {
  const assets = fs.readdirSync(path.join(cwd, srcFolder, assetsFolder));
  copyFiles(
    { from: path.join(srcFolder, assetsFolder), to: distFolder },
    ...assets
  );
}

/**
 * copies a collection of files from the source folder to the distribution folder.
 * @param { from, to } - the folder to copy from, and the folder to copy to (both strings).
 * @param  {Array<string>} files - a collection of files (strings) to be copied.
 */
function copyFiles({ from, to }, ...files) {
  files.forEach((file) => {
    fs.copyFileSync(path.join(from, file), path.join(to, file));
  });
}

/**
 * deletes the output folder and remakes it.
 */
function regenerateOutputFolder() {
  fs.rmSync(distFolder, { force: true, recursive: true });
  fs.mkdirSync(distFolder);
}

/**
 * starts clean process and prints progress logs.
 */
function clean() {
  console.log(`cleaning ${distFolder}.\n`);
  regenerateOutputFolder();
  copyFiles({ from: srcFolder, to: distFolder }, entryFile);
  copyAssets();
  console.log(`${distFolder} cleaned.\n`);
}

clean();
```

### Resources

- [Node / fs](https://nodejs.org/api/fs.html)
- [Node / path](https://nodejs.org/api/path.html)
