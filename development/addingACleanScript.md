## Adding a Clean Script

#### Last Updated: May 16, 2021

This script:

- Removes the output folder for a project (this could be something like `dist` or `build` or `lib`).
- Remakes the folder.
- Copies the entry file (`index.html`).

It can be extended to also copy any static assets to the build folder, or copy CSS or other files that may not need to be processed by a build tool like Rollup or Webpack.

<hr />

```js
import fs from "fs";
import path from "path";

const cwd = process.cwd();
const distFolder = "dist";
const srcFolder = "src";
const entryFile = "index.html";

/**
 * ### copyEntryFile
 * copies the index.html file into the build folder.
 */
function copyEntryFile() {
  fs.copyFileSync(
    path.join(cwd, srcFolder, entryFile),
    path.join(cwd, distFolder, entryFile)
  );
}

/**
 * ### regenerateOutputFolder
 * deletes the output folder and remakes it.
 */
function regenerateOutputFolder() {
  fs.rmSync(distFolder, { force: true, recursive: true });
  fs.mkdirSync(distFolder);
}

/**
 * ### init
 * starts clean process and prints progress logs.
 */
function init() {
  console.log(`cleaning ${distFolder}.\n`);
  regenerateOutputFolder();
  copyEntryFile();
  console.log(`${distFolder} cleaned.\n`);
}

init();
```

### Resources

- [Node / fs](https://nodejs.org/api/fs.html)
- [Node / path](https://nodejs.org/api/path.html)
