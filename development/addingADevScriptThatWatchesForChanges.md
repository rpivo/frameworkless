## Adding a Dev Script That Watches for Changes

```js
import { spawnSync } from "child_process";
import fs from "fs";
import path from "path";

const srcPath = path.join(process.cwd(), "src");

const options = {
  cwd: srcPath,
  stdio: "inherit",
};

let debounce = null;

/**
 * ### build
 * runs the clean script followed by the build script. clears the debounce value after these
 * processes finish.
 */
function build() {
  spawnSync("npm", ["run", "clean"], options);
  spawnSync("npm", ["run", "build"], options);
  console.log("\nwatching for changes...");
  debounce = null;
}

/**
 * ### checkDebounce
 * if debounce is falsy, sets debounce to a setTimeout that calls build() after half a second.
 */
function checkDebounce() {
  if (!debounce) debounce = setTimeout(build, 500);
}

/**
 * ### init
 * runs initial build, and then calls fs.watch() to watch the srcPath for changes. If a change
 * occurs, it calls checkDebounce.
 */
function init() {
  build();
  fs.watch(srcPath, { recursive: true }, checkDebounce);
}

init();
```
