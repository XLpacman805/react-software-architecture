# React Software Architecture #

https://www.linkedin.com/learning/react-software-architecture/

## Software Architecture Basics ##

### What is Software Architecture ###

The study of the broader structure, organization, and patterns of a adevelopment projet. Primarily the ones that impace developer productivity over time.

What's being covered

- Server Side Rendering

- State Management

- Data loading.

- Code Splitting

- Project Organization

## Server Side Rendering ##

The server returns the final HTML instead of the client taking on all the load for that. 

### Basic React SSR ###

- Need a node/expressjs server. (don't really need to use node but its useful);

- Need babel packages `npm install --save-dev @babel/core @babel/node @babel/preset-env @babel/preset-react nodemon`

- Need babelrc file. `{"presets": ["@babel/preset-env", "@babel/preset-react"]}`

server.js
```js
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';

const app = express();
app.use(express.static('./build', { index: false }));

app.get('/*', (req, res) => {
  const reactApp = renderToString(
    <h1> Hello from the server side!</h1>
  );

  return res.send(`
    <html>
      <body>
        <div id="root">
          ${reactApp}
        </div>
      </body>
    </html>
  `);
});

app.listen(8080, () => {
  console.log('server listening on port 8080');
});
```

### Building and Rendering an SSR React App ###

- Need to build project before it can be rendered. `npm run build`

- Will also need to tell express to use that build folder `app.use(express.static('./build', { index: false }));`

### Routing with Server Side Rendering ###

- Need StaticRouter, its the equivalent of BrowserRouter but for the backend. `import {StaticRouter} from 'react-router-dom'`
```js
  const reactApp = renderToString(
    <StaticRouter location={req.url}>
      <App />
    </StaticRouter>
  );
  ```

  - Move the Browser router out of the app component into the index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.hydrate(
  <React.StrictMode>
    <BrowserRouter>
      <App />
      </BrowserRouter>
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

- The responsibilies of the index.js change. It's hydrate instead of render. It'll add React to the plain HTML given from the server.

- Need to replace the root in the index.html file with the rendered react app string.

```js
app.get('/*', (req, res) => {
  const reactApp = renderToString(
    <StaticRouter location={req.url}>
      <App />
    </StaticRouter>
  );

    const templateFile = path.resolve('./build/index.html');
    fs.readFile(templateFile, 'utf8', (err, data) => {
      if (err) {
        return res.status(500).send(err);
      }

      return res.send(
        data.replace('<div id="root"></div>', `<div id="root">${reactApp}</div>`)
      );
    });
});
```

### Styling with SSR ###

- Need CSS modules to work on SSR, need to edit the webpack if you wanna go that route.

- You can also just put all the css in the index.css

- Styled components is an easy way to get encapsulated styling that can be SSR'd

  - Needs extra work to get it to SSR. The styles need to be scraped from the generated app and put into our index html.

  ```html
    <!-- ... put this unique string into the index html to replace it with styles on the server side. -->
    {{styles}}
    <title>React App</title>
  </head>
  ```

  ```js
  import { ServerStyleSheet } from 'styled-components';
  // ...
  const sheet = new ServerStyleSheet();

  const reactApp = renderToString(
    sheet.collectStyles(
      <StaticRouter location={req.url}>
        <App />
      </StaticRouter>
    )
  );

  //...
  return res.send(
        data.replace('<div id="root"></div>', `<div id="root">${reactApp}</div>`)
          .replace('{{styles}}', sheet.getStyleTags())
      );
  ```

  ### SSR Caveats ###

  - Your app executes in the server, not the browser (obviously), but that also means you cant use browser api's. No window or document objects.