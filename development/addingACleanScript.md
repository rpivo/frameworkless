## Adding a Clean Script

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
