## Implementing a Hot-Reloading Server

#### Last Updated: May 19, 2021

This hot-reloading server implementation uses the [**EventSource** Web API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) to unidirectionally send pings from the server to the client whenever the client should refresh the page. The server will send a ping whenever the source code is rebuilt.

On starting the server, the browser will automatically open at `http://localhost:8000`.

<hr />

First, you can add a `serve` script to **package.json**. This will run **Server.js** inside a **util** folder.

```json
"serve": "node ./util/Server.js",
```

The **Server.js** file is a singleton class `Server`. At the bottom of this file, an instance of `Server` is exported.

Inside the `Server` constructor, an instance of the `Watcher` class is constructed. This class extends from Node's `EventEmitter`, so it will be able to emit and listen to events. Here, you can listen to a custom event called `"refresh"`. When this event comes in, `this.shouldReload` will be set to `true`. This will signify to the `Server` that the `Watcher` has rebuilt the source code, and the `Server` can now refresh the page.

The constructor also calls `init()`, which essentially just calls `http.createServer()`. This native Node method returns an instance of `http.Server`.

When the `http.createServer()` method finishes creating an instance of `http.Server`, it will call the `openBrowser()` method callback, which opens the browser at `http://localhost:8000` to serve the application.

The callback that's passed in as an argument to `http.createServer()` is automatically registered as the callback for the `"request"` event (as a convenience of using `http.createServer()`). Any file that the client requests will cause this callback to be called.

Mostly, the internal logic for this callback handles the various files the server might request. Depending on the type of file requested (HTML/JS/CSS), the `Server` will need to respond with an appropriate `"Content-Type"` header so that the browser will know what to do with that file once it's received.

The `if (filename.endsWith("/sse")) { ... }` branch determines whether or not the client should refresh. When the client first opens, a new `EventSource` instance is created (you'll see the code for this in **Watcher.js**). This `EventSource` will send a request to the `Server` for the `/sse` URL.

`sse` here stands for [server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events). The EventSource API is like a unidirectional WebSocket where the `Server` is able to ping the client, but not the other way around. This is a useful API in this scenario since we really only want to let the client know when it should refresh the page.

When the `Server` receives the initial `/sse` connection request, it will start an interval heartbeat that occurs every 5 seconds. During each heartbeat, if `this.shouldReload` is true (which happens when the source code is rebuilt), the `Server` sends a `data: refresh` ping to the client. The `EventSource` instance in the client is listening for pings like these. When it receives this ping, it will refresh the page.

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
   * opens the default system browser and navigates to localhost at the given port.
   */
  openBrowser = () => {
    const { platform } = process;

    const openCommand =
      platform === "darwin"
        ? "open"
        : platform === "win32"
        ? "start"
        : "xdg-open";

    spawn(openCommand, [`http://localhost:${this.port}`], { stdio: "inherit" });
  };
}

export default new Server();
```

The `Watcher` class is responsible for cleaning the distribution folder, building the source code, and watching for any changes in the source code directory.

In the constructor, the **Watcher** calls a `build()` method, which runs two npm commands (the details of which aren't included here): `clean` and `build`.

These scripts can be implemented in whatever way you want; the important part is that the `clean` script should delete the distribution folder, remake it, and re-add any static files needed within the distribution folder, and the `build` script should build the source code and deposit the output in the distribution folder.

After these two commands are finished, the `injectEventSource` method is called. This adds a development-only script to the entry file (`index.html`). The script creates a new `EventSource` instance that listens for pings at `/sse`. When it receives one, it refreshes the page.

Once this initial build finishes, `fs.watch()` is called, which will watch for any changes inside of the source folder. If a change is saved, the `checkDebounce()` callback is called. This method will trigger a rebuild without multiple rebuilds being triggered in rapid succession.

After each `build()` after the first, the `Watcher` will emit a `"refresh"` event. When the `Server` receives this event, it will send a ping to the client to refresh the page.

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

    this.initialized = false;

    this.options = {
      cwd: this.srcPath,
      stdio: "inherit",
    };

    this.build();
    fs.watch(this.srcPath, { recursive: true }, this.checkDebounce);
  }

  /**
   * runs the clean script followed by the build script. calls injectEventSource() to inject the
   * EventSource js in the entry file. clears the debounce value after these processes finish.
   */
  build = () => {
    spawnSync("npm", ["run", "clean"], this.options);
    spawnSync("npm", ["run", "build"], this.options);

    this.injectEventSource();

    console.log("\nwatching for changes...");

    this.debounce = null;

    if (this.initialized) this.emit("refresh");
    else this.initialized = true;
  };

  /**
   * if debounce is falsy, sets debounce to a setTimeout that calls build() after half a second.
   */
  checkDebounce = () => {
    if (!this.debounce) this.debounce = setTimeout(this.build, 500);
  };

  /**
   * when serving locally, the client will need an EventSource instance that will receive pings
   * from the server. When it receives these pings, the client will refresh the page. This
   * function injects the js in the entry file to add the EventSource for local development.
   */
  injectEventSource = () => {
    let contents = fs.readFileSync(this.entryFile, { encoding: "utf-8" });
    const eventSourceScript = `<script>new EventSource("/sse").onmessage = () => window.location.reload();</script>`;
    contents = contents.replace(/\<\/body>/g, eventSourceScript + "</body>");
    fs.writeFileSync(this.entryFile, contents);
  };
}
```

### Resources

- [MDN / EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
- [MDN / Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [Node / Child Process](https://nodejs.org/api/child_process.html)
- [Node / Events](https://nodejs.org/api/events.html)
- [Node / File System](https://nodejs.org/api/fs.html)
- [Node / HTTP](https://nodejs.org/api/http.html)
- [Node / Path](https://nodejs.org/api/path.html)
