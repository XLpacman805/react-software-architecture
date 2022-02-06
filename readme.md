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

## State Management Architecture ##

How an application handles the data needs of its components, with regards to loading, storing, persisting, and sharing this data.

#### State Management Tech: ####

1. useState Hook
2. Context
3. Recoil
4. Redux
5. MobX

#### State management tools and techniques depend on: ####

- Size and complexity of application

- How many components need to share that data

- Unique strengths and weaknesses of each

#### Different Sizes of State ####

- Small state (isolated/atomic pieces of data)

  - useState

  - Context

- Medium state (smalled subset of components that share data)

  - Recoil

- Big state (state required by a large number of components)

  - Redux

  - MobX

### Small state with the useState Hook ###

Your normal useState hook `const [numberOfClicks, setNumberOfClicks] = useState(0);`.

### Small state with Context ###

Context lets components share state without having to pass data around as props. You can avoid prop drilling.

- Need to make a file for each of your contexts. You can import it into whatever component.

```js
// CounterContext.js
import { createContext } from 'react';
export const CounterContext = createContext();
```

- Need to make a provider file
```js
// CounterProvider.js
import { useState } from "react";
import { CounterContext } from "./CounterContext";

export const CounterProvider = ({children}) => {
  const [numberOfClicks, setNumberOfClicks] = useState(0);

  const increment = amount => {
    setNumberOfClicks(numberOfClicks + 1);
  }

  return (
    <CounterContext.Provider value={{numberOfClicks, increment}}>
      {children}
    </CounterContext.Provider>
  )
}
```

- Wrap the Context around what components need its data.
```js
//app.js
const App = () => {
	return (
			<CounterProvider>
				<h1> State Management Example </h1>
				<CounterButton />
			</CounterProvider>

	);
}
```

### Accessing context inside components ###

- Allow a child component to access the value of a context.

```js
//CounterProvider.js
import { useState } from "react";
import { CounterContext } from "./CounterContext";

export const CounterProvider = ({children}) => {
  const [numberOfClicks, setNumberOfClicks] = useState(0);

  const increment = incrementBy => {
    setNumberOfClicks(numberOfClicks + incrementBy);
  }

  return (
    <CounterContext.Provider value={{numberOfClicks, increment}}>
      {children}
    </CounterContext.Provider>
  )
}
```

```js
//CounterButton.js
import { useContext, useState } from "react"
import { CounterContext } from "./CounterContext"

export const CounterButton = () => {
  const { numberOfClicks, increment } = useContext(CounterContext)
  const [incrementBy, setIncrementBy] = useState(1);

  return (
    <>
      <p> You have clicked the button {numberOfClicks} times</p>
      <label>
        Increment By: 
        <input
          value={incrementBy}
          onChange={e => setIncrementBy(Number(e.target.value))}
          type="number"
        />
      </label>
      <button
        onClick={() => increment(incrementBy)}
      >
        Click
      </button>
    </>
  )
}
```

### Medium State with Recoil ###

- You wrap your whole app in `RecoilRoot`, it keeps track of all our states. 

- Recoil uses two concepts. "Atoms and Selectors". Atoms hold your data.

- Recoil has a `useRecoilState` hook that is very similar to `useState`. 

- Recoil shares state between components. If two components useRecoilState on the same key, its going to render the same value. Beware of this. 

- `useRecoilValue(key)` hook pretty much just gets the value. Readonly.