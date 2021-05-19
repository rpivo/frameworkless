## Adding a Serve Script

#### Last Updated: May 19, 2021

You can add a `serve` script by using Node's `createServer()` method from the `http` module.

```js
"serve": "node ./util/serve.js"
```

The below snippet calls an `init()` function, which calls `http.createServer()` and registers the `openBrowser()` callback to be called once the `listen()` method is called on the returned `http.Server` instance that will be returned from `http.createServer()`.

`createServer()` takes in a callback that is automatically registered with the `request` event. When the `request` event is emitted, this callback is called. The callback will receive an `http.IncomingMessage` request class instance and an `http.ServerResponse` response class instance.

The `http.IncomingMessage` request object is used in the callback to read the URL that was requested (`req.url`).

The `http.ServerResponse` response object is used to send response headers to the request using `res.writeHead()`, and to signal to the request that the response is complete by using `res.end()`. If the requested content is available, the content is sent as the response body while calling `res.end()`.

Based on the request URL, the response headers will update the `contentType` variable so that the `Content-Type` response header will be appropriate for the requested content, (either, `css`, `javascript`, or `html`). If this `Content-Type` header is not correctly set for each specific file, then that file will not be served correctly.

`createServer()` returns a new instance of `http.Server`. Once the `http.Server` is created, you can call `listen()` on it to listen on the given port (`8000`, in this case).

The `openBrowser()` callback is registered as the callback for `listen()`. `openBrowser()` checks the operating system and calls the appropriate open command for the OS. This will open up the default browser and navigate to localhost at the given port.

```js
import { spawn } from "child_process";
import fs from "fs";
import http from "http";
import path from "path";

const outFolder = "dist";
const port = 8000;

/**
 * opens the default system browser and navigates to localhost at the given port.
 */
function openBrowser() {
  const { platform } = process;
  spawn(
    platform === "darwin"
      ? "open"
      : platform === "win32"
      ? "start"
      : "xdg-open",
    [`http://localhost:${port}`],
    { stdio: "inherit" }
  );
}

/**
 * calls http.createServer to create an http.Server instance, and calls the openBrowser function
 * once `listen()` is called on the http.Server instance.
 */
function init() {
  // creates a new http.Server instance, and registers a callback with the `request` event.
  http
    .createServer((req, res) => {
      let filename = path.join(process.cwd(), outFolder, req.url);

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
    .listen(port, openBrowser);

  console.log(
    "\nserver running at http://localhost:" + port + "/\nCTRL + C to shutdown\n"
  );
}

init();
```

### Resources

- [Node / Child Process](https://nodejs.org/api/child_process.html)
- [Node / File System](https://nodejs.org/api/fs.html)
- [Node / HTTP](https://nodejs.org/api/http.html)
- [Node / Path](https://nodejs.org/api/path.html)
