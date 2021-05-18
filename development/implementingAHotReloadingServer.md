## Implementing a Hot-Reloading Server

#### Last Updated: May 18, 2021

**clean.js**

```js
import fs from "fs";
import path from "path";

const cwd = process.cwd();

const assetsFolder = "assets";
const distFolder = "dist";
const srcFolder = "src";
const entryFile = "index.html";

/**
 * ### copyAssets
 * copies each asset from a given assets folder to the output folder.
 */
function copyAssets() {
  const assets = fs.readdirSync(path.join(cwd, srcFolder, assetsFolder));

  assets.forEach((asset) =>
    fs.copyFileSync(
      path.join(cwd, srcFolder, assetsFolder, asset),
      path.join(cwd, distFolder, asset)
    )
  );
}

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
 * ### clean
 * starts clean process and prints progress logs.
 */
function clean() {
  console.log(`cleaning ${distFolder}.\n`);
  regenerateOutputFolder();
  copyEntryFile();
  copyAssets();
  console.log(`${distFolder} cleaned.\n`);
}

clean();
```

**Server.js**

```js
import { spawn } from "child_process";
import fs from "fs";
import http from "http";
import path from "path";
import Watcher from "./Watcher.js";

class Server {
  constructor() {
    this.outFolder = "dist";
    this.port = 8000;
    this.shouldReload = false;
    this.sseInterval = null;

    this.watcher = new Watcher().on("refresh", () => {
      this.shouldReload = true;
    });

    this.init();
  }

  /**
   * ### init
   * calls http.createServer to create an http.Server instance, and calls the openBrowser function
   * once `listen()` is called on the http.Server instance.
   */
  init = () => {
    // creates a new http.Server instance, and registers a callback with the `request` event.
    http
      .createServer((req, res) => {
        let filename = path.join(process.cwd(), this.outFolder, req.url);

        // if requested url ends with /sse, then this is the server-sent event connection. We'll
        // set up a five-second interval to check if the app has been rebuilt. If it has, send a
        // message to the client EventSource so that it knows to refresh the page.
        if (filename.endsWith("/sse")) {
          if (this.sseInterval) clearInterval(this.sseInterval);

          this.sseInterval = setInterval(() => {
            if (this.shouldReload) {
              res.writeHead(200, {
                "Content-Type": "text/event-stream",
              });
              res.write("data: refresh\n\n");

              this.shouldReload = false;
            }
          }, 5000);

          return;
        }

        // if the req.url is a file path, we can assume the request is for the index.html file.
        try {
          const stats = fs.statSync(filename);
          if (stats && stats.isDirectory()) filename += "index.html";
        } catch (error) {
          console.error("There was an error when trying to read a file.", {
            error,
            filename,
          });
          return;
        }

        // check what the req.url ends with, and set the Content-Type response header accordingly.
        // read the contents of the requested file and send these contents as the response body.
        try {
          const contents = fs.readFileSync(filename, { encoding: "binary" });

          let contentType = "";

          if (filename.endsWith(".css")) contentType = "css";
          else if (filename.endsWith(".js")) contentType = "javascript";
          else contentType = "html";

          res.writeHead(200, { "Content-Type": `text/${contentType}` });
          res.end(contents, "binary");
        } catch (e) {
          res.writeHead(500, { "Content-Type": "text/plain" });
          res.end(e + "\n");
        }
      })
      // calls listen() on the http.Server instance, listening for requests at the given port. Passes
      // in the openBrowser function as a callback.
      .listen(this.port, this.openBrowser);

    console.log(
      "\nserver running at http://localhost:" +
        this.port +
        "/\nCTRL + C to shutdown\n"
    );
  };

  /**
   * ### openBrowser
   * opens the default system browser and navigates to localhost at the given port.
   */
  openBrowser = () => {
    const { platform } = process;
    spawn(
      platform === "darwin"
        ? "open"
        : platform === "win32"
        ? "start"
        : "xdg-open",
      [`http://localhost:${this.port}`],
      { stdio: "inherit" }
    );
  };
}

new Server();
```

**Watcher.js**

```js
import { spawnSync } from "child_process";
import EventEmitter from "events";
import fs from "fs";
import path from "path";

export default class Watcher extends EventEmitter {
  constructor() {
    super();

    const cwd = process.cwd();

    this.entryFile = path.join(cwd, "dist/index.html");
    this.srcPath = path.join(cwd, "src");

    this.options = {
      cwd: this.srcPath,
      stdio: "inherit",
    };

    this.build();
    fs.watch(this.srcPath, { recursive: true }, this.checkDebounce);
  }

  /**
   * ### build
   * runs the clean script followed by the build script. calls injectEventSource() to inject the
   * EventSource js in the entry file. clears the debounce value after these processes finish.
   */
  build = () => {
    spawnSync("npm", ["run", "clean"], this.options);
    spawnSync("npm", ["run", "build"], this.options);

    this.injectEventSource();

    console.log("\nwatching for changes...");

    this.debounce = null;

    this.emit("refresh");
  };

  /**
   * ### checkDebounce
   * if debounce is falsy, sets debounce to a setTimeout that calls build() after half a second.
   */
  checkDebounce = () => {
    if (!this.debounce) this.debounce = setTimeout(this.build, 500);
  };

  /**
   * ### injectEventSource
   * when serving locally, the client will need an EventSource instance that will receive pings
   * from the server. When it receives these pings, the client will refresh the page. This
   * function injects the js in the entry file to add the EventSource for local development.
   */
  injectEventSource = () => {
    let contents = fs.readFileSync(this.entryFile, { encoding: "utf-8" });
    const eventSourceScript = `<script>const sse = new EventSource("/sse"); sse.onmessage = () => window.location.reload();</script>`;
    contents = contents.replace(/\<\/body>/g, eventSourceScript + "</body>");
    fs.writeFileSync(this.entryFile, contents);
  };
}
```

### Resources

- [Node / Child Process](https://nodejs.org/api/child_process.html)
- [Node / Events](https://nodejs.org/api/events.html)
- [Node / File System](https://nodejs.org/api/fs.html)
- [Node / HTTP](https://nodejs.org/api/http.html)
- [Node / Path](https://nodejs.org/api/path.html)
