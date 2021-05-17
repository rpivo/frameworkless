## Adding a Dev Script That Watches for Changes

#### Last Updated: May 16, 2021

You can add a `dev` script that watches for changes in a specified source code folder by using the `watch` method from Node's `fs` module.

<hr />

First, you can add a `dev` script like so inside the **package.json** scripts:

```json
"dev": "node ./util/dev.js"
```

The `dev` script below runs `build` function, which serially runs the `clean` script followed by the `build` script. It then sets the outer scope `debounce` variable to null.

After all this, `fs.watch()` is called, which watches the source code path (`srcPath`). Inside the second options argument for `fs.watch()`, you can specify the `recursive` property as `true`, which will allow the watch method to watch any file within this source code path.

The third argument for `fs.watch()` is the callback that will be called when the watch method sees that a file has changed. The callback here is `checkDebounce()`.

`checkDebounce()` checks to see if the outer scope `debounce` variable is falsy. If it is, it sets `debounce` to a `setTimeout` that expires after half a second. Once it expires, it calls the `build` function. This debounce prevents the `build` function from being called in rapid succession.

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

### Resources

- [Node / child Process](https://nodejs.org/api/child_process.html)
- [Node / File System](https://nodejs.org/api/fs.html)
- [Node / Path](https://nodejs.org/api/path.html)
