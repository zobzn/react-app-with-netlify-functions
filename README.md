Init project and install some dependencies (and switch back from yarn to npm)

```bash
npm init react-app netlify-react-app
cd netlify-react-app
rm -rf ./node_modules/ yarn.lock
npm i && npm up
npm i -D npm-run-all prettier netlify-lambda
```

Update scripts section in package.json

```json
"scripts": {
  "build": "run-p build:**",
  "build:app": "react-scripts build",
  "build:functions": "netlify-lambda build functions/",
  "start": "run-p start:**",
  "start:app": "react-scripts start",
  "start:functions": "netlify-lambda serve functions/",
  "test": "react-scripts test",
  "prettier": "prettier --write {./*,./src/**/*,./functions/**/*}.{js,jsx,json,md}"
}
```

Create a netlify.toml so that netlify knows where the build is

```toml
[build]
  command = "npm run build"
  functions = "build-lambda" # netlify-lambda gets build to this folder
  publish = "build"  # create-react-app builds to this folder
```

Remember to add build-lambda to .gitignore

```bash
echo "" >> .gitignore
echo "#for netlify" >> .gitignore
echo "/build-lambda" >> .gitignore
```

Create your first function by creating a JS file in functions folder

```bash
mkdir functions
echo "export async function handler(event) {" >> functions/hello.js
echo "  return {" >> functions/hello.js
echo "    statusCode: 200," >> functions/hello.js
echo "    body: JSON.stringify({ data: 'hello world' })" >> functions/hello.js
echo "  }" >> functions/hello.js
echo "}" >> functions/hello.js
```

To serve functions run `npm run serve:lambda`.
From the frontend application you can now use fetch to query `http://localhost:9000/.netlify/functions/hello`

Configure proxy for functions with `src/setupProxy.js` to be able to call functions with `/.netlify/functions/hello`

```javascript
const proxy = require("http-proxy-middleware");
module.exports = function(app) {
  app.use(
    proxy("/.netlify/functions/", {
      target: "http://localhost:9000/",
      pathRewrite: {
        "^/\\.netlify/functions": ""
      }
    })
  );
};
```

Material is based on:

- https://dev.to/boywithsilverwings/creating-a-jamstack-reader-app-with-react-netlify-functions-ckj
- https://create-react-app.dev/docs/setting-up-your-editor/
