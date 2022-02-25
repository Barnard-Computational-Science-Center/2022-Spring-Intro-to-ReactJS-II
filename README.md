## Introduction to ReactJS Part II - Spring 2022


##### Prerequisites

- [Complete part I of this series](https://github.com/Barnard-Computational-Science-Center/2022-Spring-Intro-to-ReactJS)
 - [Install NodeJS](https://nodejs.org/en/)
 - [Install and configure Tailwind CSS](https://tailwindcss.com/docs/guides/create-react-app)
 - Bootstrap a React app with `create-react-app`


##### Goals

- This workshop builds on the Introduction to ReactJS workshop and will explore data-fetching, side-effects, and techniques for handling asynchronous operations with async/await.

##### Resources

Water Consumption in New York City JSON Data

```
https://data.cityofnewyork.us/resource/ia2d-e54m.json
```

New York City Restaurant Inspection Results

```
https://data.cityofnewyork.us/resource/43nn-pn8j.json
```

---

### 1. Introduction

Make sure you have a completed [part I]((https://github.com/Barnard-Computational-Science-Center/2022-Spring-Intro-to-ReactJS)) of this series, as this guide starts where that guide ends. To get quickly caught up, you should:

- Bootstrap a React app with `create-react-app`.
- Install and configure Tailwind CSS
- Copy/paste the last block of code from the prior workshop into your `App.js` file.
- If you don't have the `data.json` file, you can remove the import and replace any references to `data` with an empty array: `const [cards, setCards] = useState([]);`

Start your app with:

```
npm run start
```

Once you're setup, `App.js` should have two components, `App.js` and `Card`. We will move `Card` into its own file (`Card.js`), then import into `App.js`:


```javascript
import React, { useState } from "react";
import Card from "./Card"
import data from "./data.json";

function App() {
  const [cards, setCards] = useState(data);

  function handleClick(year) {
    const filteredCards = cards.filter((card) => card.year !== year);
    setCards(filteredCards);
  }

  return (
    <div className="bg-red-50">
      <h3 className="mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        Water Consumption in New York City
      </h3>

      <ul className="grid grid-cols-8 gap-4 px-10 ">
        {cards.map((obj) => (
          <Card handleClick={handleClick} obj={obj} />
        ))}
      </ul>
    </div>
  );
}

export default App;

```


#### 1.1 App.js
- We're currently using hard-coded data (`data.json`) that we copy/pasted from [NYC Open Data JSON endpoint](https://data.cityofnewyork.us/resource/ia2d-e54m.json).
- We will replace this with a network request using the browser's [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) API.
- In addition to `useState`, add `useEffect` to import statement.
- Remove the `data` import statement.
- Set the initial state to an empty array: `useState([])` instead of `useState(data)`.
- Declare a variable for data url: `const url = "...."`


~~~~javascript
import React, { useState, useEffect } from "react";
import Card from "./Card"

const url = "https://data.cityofnewyork.us/resource/ia2d-e54m.json"

function App() {
  const [cards, setCards] = useState([]);

  function handleClick(year) {
    const filteredCards = cards.filter((card) => card.year !== year);
    setCards(filteredCards);
  }

  return (
    <div className="bg-red-50">
      <h3 className="mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        Water Consumption in New York City
      </h3>

      <ul className="grid grid-cols-8 gap-4 px-10 ">
        {cards.map((obj) => (
          <Card handleClick={handleClick} obj={obj} />
        ))}
      </ul>
    </div>
  );
}

export default App;
~~~~




#### 1.2 App.js with `useEffect`
- The `useEffect` hook lets us ["perform side effects in function components."](https://reactjs.org/docs/hooks-effect.html)
- A function has a *side effect* when it affects, performs operations on, or depends on something outside of itself (outside its own "scope").
- The `useEffect` hook allows us to perform side effects (either in response to events, like the click of a button, or as stand-alone, one-time events). Fetching data from a remote resource is a common side effect.

```javascript
import React, { useState, useEffect } from "react";
import Card from "./Card"

const url = "https://data.cityofnewyork.us/resource/ia2d-e54m.json"

function App() {
  const [cards, setCards] = useState([]);

  function handleClick(year) {
    const filteredCards = cards.filter((card) => card.year !== year);
    setCards(filteredCards);
  }

  useEffect(() => {
    console.log("Hello world!")
  });

  return (
    <div className="bg-red-50">
      <h3 className="mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        Water Consumption in New York City
      </h3>

      <ul className="grid grid-cols-8 gap-4 px-10 ">
        {cards.map((obj) => (
          <Card handleClick={handleClick} obj={obj} />
        ))}
      </ul>
    </div>
  );
}

export default App;
```


#### 1.3 Additional details on `useEffect`

- What happens if we update the state inside the effect hook? (Not recommended!)

```javascript
  useEffect(() => {
    //console.log("Hello world!")
    setCards([])
  });
```

Why do we get stuck in an infinite loop? From the React docs, effects run "both after the first render and after every update".

Since React rerenders on each state update, and the effect hook renders after every update, by updating the state inside the effect, we get stuck in the loop:

1. The initial render triggers the effect
2. Inside `useEffect` we update the state
3. Since the state updated, React rerenders
4. Rerendering triggers the effect
5. Inside `useEffect` we update the state
6. ...
7. ∞

The prevent this, we pass an empty dependency array to the effect:

```javascript
  useEffect(() => {
    //console.log("Hello world!")
    setCards([])
  }, []); // Empty dependency array.
```

Usually, we want the effect to run in response to a change in a value. To do so, we can pass that value (or values) to the dependency array. 


```javascript
  useEffect(() => {
    // Do stuff...
  }, [count]); // This effect executes only when "count" changes.
```

Let's reintroduce the `count` variable from part I, and use it in the effect:

- Declare a new state variable & setter: `count` & `setCount`.
- Add a button to increment the count.
- Add an effect hook with `count` inside the dependency array.
- Console log whether the count is even or odd.


```javascript
import React, { useState, useEffect } from "react";
import Card from "./Card";

const url = "https://data.cityofnewyork.us/resource/ia2d-e54m.json";

function App() {
  const [cards, setCards] = useState([]);

  const [count, setCount] = useState(0);

  function handleClick(year) {
    const filteredCards = cards.filter((card) => card.year !== year);
    setCards(filteredCards);
  }

  useEffect(() => {
    if (count % 2 === 0) {
      console.log(count + " is even.");
    } else {
      console.log(count + " is odd.");
    }
  }, [count]);

  return (
    <div className="bg-red-50">
      <h3 className="mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        Water Consumption in New York City
      </h3>

      <button className="border p-2 m-2" onClick={() => setCount(count + 1)}>
        Click me
      </button>

      {count}

      <ul className="grid grid-cols-8 gap-4 px-10 ">
        {cards.map((obj) => (
          <Card handleClick={handleClick} obj={obj} />
        ))}
      </ul>
    </div>
  );
}

export default App;



```

 
### 2. Data Fetching, Asynchronous Programming, Promises

Let's fetch the json data from the NYC Open Data API, and set it as our state. We can do this inside an effect. Since we only need to fetch the data once, we can pass an empty dependency array to the effect. Also, console log `data`:

```javascript
  useEffect(() => {
    const data = fetch(url)
    console.log(data)
    setCards(data)
  }, []);
```

We were expecting `data` to be an array of objects:

```javascript
[
  {
    year: "1979",
    new_york_city_population: "7102100",
    nyc_consumption_million_gallons_per_day: "1512",
    per_capita_gallons_per_person_per_day: "213",
  },
  {
    year: "1980",
    new_york_city_population: "7071639",
    nyc_consumption_million_gallons_per_day: "1506",
    per_capita_gallons_per_person_per_day: "213",
  }, ...
]
```

But instead we got back a `Promise`:

```
Promise {<pending>}
```

From the [documentation](https://developer.mozilla.org/en-US/docs/Web/API/fetch), fetch returns a "promise that resolves to a Response object", and the [`Promise` object represents the eventual completion (or failure) of an asynchronous operation and its resulting value.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

To make sense of this, we have to understand that the browser is an asynchronous programming environment. Loosely speaking, many things can happen at the same time, and things don't need to necessarily finish before our program moves onto the next thing. 

An easy way to think about this is to consider what would happen if it weren't the case. That is, if only one thing could happen at one time. Suppose you visited the [New York Times](www.nytimes.com) website and some JavaScript code was taking a little while to execute. In a synchronous environment, the browser would freeze in place until that code finished.

There are essentially three ways to deal with asynchronous operations in JavaScript: **callbacks**, **promises**, and **async/await** (the third paradigm, async/await, is mostly a syntactic abstraction over promises that makes them easier to use).

While the details of these paradigms are outside of the scope of this workshop, we can fix our code with a simple promise chain:

```javascript
  useEffect(() => {
    const data = fetch(url)
      .then(res => res.json())
      .then(json => setCards(json))
  }, []);
```

Initially, `fetch` returns a `promise`, which will either be resolved or rejected. Since neither has happened yet, it's in an intermediate state. If `fetch` is successful, the `then` block will execute, and it too will return a `promise`. 

It's important to remember that promises **do not block** the rest of the program from executing (they are placed in a queue).

While we can continue to use promises in this way, `.then()` chaining can get messy. We can use async/await to clean up some of this code. 



#### 2.1 Async/Await
- Remove the counter variables & button.
- Declare an async function `fetchData` with the `async` keyword
- Within `fetchData` we can use the `await` keyword to ["enable asynchronous, promise-based behavior to be written in a cleaner style, avoiding the need to explicitly configure promise chains."](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- With the `await` keyword, we have immediate access to the value, and can use it in the next line without having to chain `.then()` statements together. 
- Async functions are non-blocking. When JavaScript reaches the `await` statement, it will not execute the rest of the function until the promise has resolved. While that's happening, other code is executed.

```javascript
import React, { useState, useEffect } from "react";
import Card from "./Card";

const url = "https://data.cityofnewyork.us/resource/ia2d-e54m.json";

async function fetchData(url, setState) {
  const response = await fetch(url);
  const jsonData = await response.json();
  setState(jsonData);
}

function App() {
  const [cards, setCards] = useState([]);

  function handleClick(year) {
    const filteredCards = cards.filter((card) => card.year !== year);
    setCards(filteredCards);
  }

  useEffect(() => {
    fetchData(url, setCards);
  }, []);

  return (
    <div className="bg-red-50">
      <h3 className="mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        Water Consumption in New York City
      </h3>
      
      <ul className="grid grid-cols-8 gap-4 px-10 ">
        {cards.map((obj) => (
          <Card handleClick={handleClick} obj={obj} />
        ))}
      </ul>
    </div>
  );
}

export default App;
```

#### 2.2 App.js - Remote Data Fetching

- Remove or comment out `<Card />` component within the return statement.
- Add `overflow-scroll` to the root `div`, and some y-margin (`my-10`) to the `h3`. You can also replace the `h3` with a `div` and make it a `flex` container.
- Declare a variable called `dataMap` whose properties are unique integer ids, 1 & 2. Each property returns an object with some metadata: `label` & `url`
- Declare a new state variable and setter called `selected` & `setSelected`
- Add a couple of buttons, and attach `setSelected` to the `onClick` event, passing the respective `dataMap` ids as parameters.
- Inside the effect, use `selected` to access the `url` property: `dataMap[selected].url` or `dataMap[selected]["url"]`. Fetch the data with this url, and add `selected` to the effect's dependency array. 


```javascript
import React, { useState, useEffect } from "react";
import Card from "./Card";

const dataMap = {
  1: {
    label: "Water Consumption in New York City",
    url: "https://data.cityofnewyork.us/resource/ia2d-e54m.json",
  },
  2: {
    label: "New York City Restaurant Inspection Results",
    url: "https://data.cityofnewyork.us/resource/43nn-pn8j.json",
  },
};

async function fetchData(url, setState) {
  const response = await fetch(url);
  const jsonData = await response.json();
  setState(jsonData);
}

function App() {
  const [selected, setSelected] = useState(1);
  const [data, setData] = useState([]);

  function handleClick(year) {
    const filteredCards = data.filter((card) => card.year !== year);
    setData(filteredCards);
  }

  useEffect(() => {
    const url = dataMap[selected].url;
    fetchData(url, setData);
  }, [selected]);

  return (
    <div className="bg-red-50 h-screen overflow-scroll">
      <div className="flex my-10 mx-auto max-w-sm border border-2 border-gray-800 bg-lime-300 px-4 py-2 text-center text-lg font-medium text-gray-900">
        <button onClick={() => setSelected(1)}>
          Water Consumption in New York City
        </button>

        <button onClick={() => setSelected(2)}>Inspection Results</button>
      </div>

      {data.map((obj) => (
        <li handleClick={handleClick} obj={obj}>
          Item...
        </li>
      ))}
    </div>
  );
}

export default App;
```

#### 2.3 InspectionCard.js

- Create a new file called `InspectionCard.js` by copy/pasting the original `Card` component. Don't forget to update the name to `InspectionCard`.
- Update the properties within the `return` body. There are many to choose from, but `dba`, `grade`, `score`, `violation_description`, and `camis` (unique id) might work better than others. 
- You can remove the `button` as well, along with the `handleClick` prop.


```javascript
export default function InspectionCard({ obj }) {
    return (
      <li className="col-span-8 my-2 border-2 border-gray-800 md:col-span-2 ">
        <div className="flex items-center justify-between bg-cyan-800">
          <h3 className="border-b-2 border-gray-800  p-2 text-lg font-semibold text-white">
            {obj.dba}
          </h3>
        </div>
  
        <div className="bg-red-200">
          <div className="flex items-center justify-between p-4 hover:bg-red-300">
            <p className="truncate font-medium text-gray-600">Score</p>
            <div className="font-medium text-gray-800 underline">{obj.score}</div>
          </div>
  
          <div className="flex items-center justify-between p-4 hover:bg-red-300">
            <p className="truncate font-medium text-gray-600">
              Grade
            </p>
            <div className="font-medium text-gray-800 underline hover:bg-red-300">
              {obj.grade}
            </div>
          </div>
        </div>
      </li>
    );
  }
```



#### 2.4 App.js & Conditional Rendering

- Import both `Card` & `InspectionCard`
- Within the return body, conditionally render the card that matches the selected data. 


#### 2.5 InspectionCard.js & Template Literals

- Template literals allow us to embed expressions into strings.
- Declare two variables `spin` and `bgColor`, and evaluate them based on the `grade` value.
- Update the `classNames` to template literals:

```javascript
className={`string text goes here ${expression}`}
```

```javascript
export default function InspectionCard({ obj }) {
    const spin = obj.grade === "A" ? "animate-spin" : "";
    const bgColor =
      obj.grade === "A"
        ? "bg-gradient-to-r from-purple-300 via-cyan-700 to-lime-500"
        : "bg-cyan-800";
  
    return (
      <li
        className={`col-span-8 my-2 border-2 border-gray-800 md:col-span-2  ${spin}`}
      >
        <div className={`flex items-center justify-between ${bgColor}`}>
          <h3 className="border-b-2 border-gray-800  p-2 text-lg font-semibold text-white">
            {obj.dba}
          </h3>
        </div>
  
        <div className="bg-red-200">
          <div className="flex items-center justify-between p-4 hover:bg-red-300">
            <p className="truncate font-medium text-gray-600">Score</p>
            <div className="font-medium text-gray-800 underline">{obj.score}</div>
          </div>
  
          <div className="flex items-center justify-between p-4 hover:bg-red-300">
            <p className="truncate font-medium text-gray-600">Grade</p>
            <div className="font-medium text-gray-800 underline hover:bg-red-300">
              {obj.grade}
            </div>
          </div>
        </div>
      </li>
    );
  }
```


#### 3. Build & Deployment

To create a production build of your app, return to the terminal and run:

	npm run build

- You can find more [details in the CRA docs](https://create-react-app.dev/docs/production-build/).
- Information about [deploying the app on different platforms](https://create-react-app.dev/docs/deployment/).
- Drag and drop the [build folder on Netifly](https://app.netlify.com/drop).
