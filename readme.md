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
