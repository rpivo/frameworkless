## Adding a Serve Script

#### Last Updated: May 16, 2021

You can add a `serve` script by using Node's `createServer()` method from the `http` module.

```js
"serve": "node ./util/serve.js"
```

`createServer()` takes in a callback that is automatically registered with the `request` event. When the `request` event is emitted, this callback is called. The callback will receive an `http.IncomingMessage` request class instance and an `http.ServerResponse` response class instance.

The `http.IncomingMessage` request object is used in the callback to read the URL that was requested (`req.url`).

The `http.ServerResponse` response object is used to send response headers to the request using `res.writeHead()`, and to signal to the request that the response is complete by using `res.end()`. If the requested content is available, the content is sent as the response body while calling `res.end()`.

Based on the request URL, the response headers will update the `contentType` variable so that the `Content-Type` response header will be appropriate for the requested content, (either, `css`, `javascript`, or `html`).

`createServer()` returns a new instance of `http.Server`. Once the `http.Server` is created, you can call `listen()` on it to listen on the given port (`8000`, in this case).

```js
import fs from "fs";
import http from "http";
import path from "path";

const outFolder = "dist";
const port = 8000;

http
  .createServer((req, res) => {
    let filename = path.join(process.cwd(), outFolder, req.url);

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
  .listen(port);

console.log(
  "\nserver running at http://localhost:" + port + "/\nCTRL + C to shutdown\n"
);
```

### Resources

- [Node / File System](https://nodejs.org/api/fs.html)
- [Node / HTTP](https://nodejs.org/api/http.html)
- [Node / Path](https://nodejs.org/api/path.html)
