## Initializing a Web App from index.html

The below example is one way to initialize a web app from inside an **index.html** file. This method uses ES Module syntax to import the main app code from an **index.js** file that's in the same directory as the **index.html** file. The main js code initializes the app by constructing a new `App` class instance.

<hr />

The initialization code should be wrapped in a `<script>` tag at the end of the body. This `<script>` tag should have a `type` attribute set to `"module"` to indicate that it's an ES module. Note that ES modules are only available in modern browsers.

In this `<script>` tag, import the main js code with `import App from "./index.js"`.

Then, listen to the `"DOMContentLoaded"` event. This event will fire once the HTML document has been loaded and parced. Once this `document` emits this event, call `new App()` to initialize the main `App` class instance for the app..

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <link href="index.css" rel="stylesheet" />
    <title>Web App</title>
  </head>
  <body>
    <main></main>
    <script type="module">
      import App from "./index.js";
      document.addEventListener("DOMContentLoaded", () => new App());
    </script>
  </body>
</html>
```

### Resources

- [MDN / Window: DOMContentLoaded Event](https://developer.mozilla.org/en-US/docs/Web/API/Window/DOMContentLoaded_event)
- [MDN / JavaScript Modules / Applying the module to your HTML](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#applying_the_module_to_your_html)
