---
title: Create a Weather App with React Hooks Part 2 
date: "2020-11-14T23:46:37.121Z"
---

For the first part of the project, we have used `useState` hook, fetched our data and console logged the data for any city typed by the user. 

Before we continue, we need to remove some of the code from the last part to be able to use our custom hook. Let's go inside our `CitySelector` component and remove `onSearch` function, also remove the `results` state. We will handle our button click from `App.js`, pass as a prop inside our Button component.

Now, my `CitySelector.js` is looking like this.

```javascript
// components/CitySelector.js

import React, {useState} from 'react';
import {Row, Col, FormControl, Button} from 'react-bootstrap';

const CitySelector = ({onSearch}) => {
    const [city, setCity] = useState('');
  
    return (
      <>
        <Row>
          <Col>
            <h1>Search your city</h1>
          </Col>
        </Row>
  
        <Row>
          <Col xs={4}>
            <FormControl
              placeholder="Enter city"
              onChange={(event) => setCity(event.target.value)}
              value={city}
            />
          </Col>
        </Row>
  
        <Row>
          <Col>
           {/* don't forget to edit our function  */}
            <Button onClick={() => onSearch(city)}>Check Weather</Button>
          </Col>
        </Row>
      </>
    );
  };

export default CitySelector;

```

Now, we will show the data in our UI and display 5 days of data. To be able to do that, we will use another hook named `useEffect` hook.

## `useEffect` Hook

The `useEffect` Hook can help us to replace React lifecycle events. Lifecycle events are a set of events that take place at some point when a component is updated, changed, or removed. These are `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. It's used for side effects (all things which happen outside of React) like network requests, managing subscriptions, DOM manipulation, setting up event listeners, timeouts, intervals, or local storage, etc.


### Short Info About Lifecycle Events

- **ComponentDidMount** is going to be called right after our component is added to the DOM. It fetches data at the initial render phase.
- **ComponentDidUpdate**, updates the DOM when anything changes like state changes.
- **ComponentWillUnmount** allows us to do any sort of cleanup. For example, if you want to invalidate a timer, or if you want to do any cleanup of any nodes, you can do that with this event. It runs right before the component is being removed from the web page.

### How `useEffect` works?

- `useEffect` listens for any change in our app.
- It takes a function and two arguments. 
- First argument helps us tell `useEffect when the code to be executed. 
- Second argument or the dependency array controls when the code gets executed. For the second argument, we can pass an array, an array with value/values, or no array at all.
  - If we don't pass an array, this will run only at initial render only once.
  - If we pass an empty array, this will run at initial render and whenever it rerenders.
  - If we pass an array with value/values inside of it, this will run at initial render and runs whenever our data changes inside the array.


## Custom Hooks for Search

Create a new folder under `src` named `hooks` then create a new file named `UseFetch.js`.

Generally, in our custom hooks, we place our logic like we can put our `useState` and `useEffect` hooks. As you can see, we have imported `useEffect` hook from React, and we have defined some `useState` variables.

For the `useEffect` hook, we have created an anonymous function. The most important part of our custom is `return` statement. Here, we return whatever we want another component to have access to. We can return an `array` or an `object`. If you return an array, we can name the returned values whatever we want outside the file. We don't need to keep the same name as we have returned. 

Another side note is about `url`. We have to define a state hook because whenever our user searches for any city, our url will change. To keep track of its state, we added a state for that. 

Also, you should take note of our `useEffect` dependency array. If we place some variables inside our array, our app will be updated whenever our URL changes. That's why we also return our `setUrl` function. 

But we may have a problem here, when we first load our app, we may not have any url, for that we have added a conditional check.
  
```javascript

// hooks/UseFetch.js

import {useState, useEffect} from 'react';

const UseFetch = (initialUrl) => {
  // create state variables
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(null);
  const [url, setUrl] = useState(initialUrl);

  useEffect(() => {
 
    setIsLoading(true);

    fetch(url)
      // check if we have an url for the initial render
       if(!url) return;
      .then((response) => response.json())
      .then((data) => {
        setIsLoading(false);
        setData(data);
      })
      .catch((error) => {
        setIsLoading(false);
        setError(error);
      });
      // dependency array 
  }, [url]);

  return { data, error, isLoading, setUrl };
};

export default UseFetch;
```

Now, let's import this into our `App.js` component, and pass our custom hook. To do this, we can destructure our variables from `UseFetch` function.

`import UseFetch from '../hooks/UseFetch';`

With the help of our custom hook, we can call our API every time the button is clicked.

```javascript
// App.js

import React from 'react';
import CitySelector from './components/CitySelector';
import './App.css';
import {Container} from 'react-bootstrap';
import UseFetch from './hooks/UseFetch'
import {API_KEY, API_BASE_URL} from './apis/config';

const App = () => {
  // destructure the returned values
  const {data, error, isLoading, setUrl} = UseFetch();

  return (
    <Container className="App">
        <CitySelector onSearch={(city) => setUrl(`${API_BASE_URL}/data/2.5/forecast?q=${city}&appid=${API_KEY}`)} />
    </Container>
  );
};

export default App;

```

Now, we can fetch our data with `useEffect` custom hook. It prints `null` multiple times because we have different setters inside our custom hook.


![use-effect.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1605337695605/VeCrGt6Zg.gif)


## Populate the Data

Now, let's populate our data and show 5 days weather data. For this, we will create another component. Under the components folder, create `WeatherList.js` component. 

```javascript

// components/WeatherList.js

import React from 'react'
import { Col, Row } from 'react-bootstrap'
import WeatherCard from './WeatherCard'

const WeatherList = ({weathers}) => {
    return (
        <Row>
           {weathers.map(({dt, main, weather}) => (
                <Col key={dt}>
                    <WeatherCard 
                    temp_max={main.temp_max} 
                    temp_min={main.temp_min} 
                    dt={dt * 1000} 
                    main={weather[0].main} 
                    icon={weather[0].icon} 
                  />
                </Col>
            ))} 
        </Row>
    )
}

export default WeatherList;
```

Now, let’s break down the code above to explain what we’ve added and how it works.

- We passed `weathers` prop and pass it from our `App.js` file. 
- For the jsx, we used `Row` and `Col` components from react-bootstrap.
- To create columns, we mapped over our weathers array and populate 5 columns next to each other displaying weather data for 5 consecutive days.
- Every column contains datetime, main and weather data from our API. 
- Nested our `WeatherCard` component inside the `WeatherList` component and pass its prop values from here.
- As you may notice, we also passed the `key` property for our mapped weather cards. If we don't pass a key, React will complain about it. Because, when we map over an array, we need an identifier like an id.

Now, we can import our `WeatherList` component inside `App.js`. Here we need to render `WeatherList` conditionally, if we have data from our API, render the `WeatherList` component, also pass our prop named `weathers` to reach our API results.

```javascript
import React from 'react';
import CitySelector from './components/CitySelector';
import './App.css';
import {Container} from 'react-bootstrap';
import UseFetch from './hooks/UseFetch';
import {API_KEY, API_BASE_URL} from './apis/config'
import WeatherList from './components/WeatherList';

const App = () => {
  const {data, error, isLoading, setUrl} = UseFetch();
  console.log(data);

  return (
    <Container className="App">
      <CitySelector onSearch={(city) => setUrl(`${API_BASE_URL}/data/2.5/forecast?q=${city}&cnt=5&appid=${API_KEY}`)} />
          
    {/* conditionally render  */}
      {data && <WeatherList weathers={data.list} />}
    </Container>
  );
};

export default App;
```


![results.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1605337768969/sns1fGOnm.png)

If you had a problem with the styling, remove inline style (style={{width: '18rem'}}) from `WeatherCard` component.

## Error Handling and Loading

As you may notice we didn't make use of `isLoading` and `error` variables yet. 

For that, we will create multiple conditional checks before rendering our `WeatherList` component. If we pass all the checks we will display our `WeatherList` component.

```javascript

// App.js
const App = () => {
  const {data, error, isLoading, setUrl} = UseFetch();

// error handling and loading
  const getContent = () => {
    if(error) return <h2>Error when fetching: {error}</h2>
    if(!data && isLoading) return <h2>LOADING...</h2>
    if(!data) return null;
    return <WeatherList weathers={data.list} />
  };

  return (
    <Container className="App">
      <CitySelector onSearch={(city) => setUrl(`${API_BASE_URL}/data/2.5/forecast?q=${city}&cnt=5&appid=${API_KEY}`)} />

      {/* don't forget the change */}
      {getContent()}
    </Container>
  );
};

export default App;
```

If we make different searches, our search is not updating. To clear the previous searches from our state, we need to edit our `UseFetch` function.

If the user types sth other than a city, I mean if it doesn't exist in our API data, we get an error page. To fix that, we will make a check if we get a `data.cod` bigger than 400 we will show an error.

```javascript

// hooks/UseFetch.js

import {useState, useEffect} from 'react';

const UseFetch = (initialUrl) => {
  // create state variables
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(null);
  const [url, setUrl] = useState(initialUrl);

  useEffect(() => {
    if(!url) return;
    setIsLoading(true);
    // clear old search
    setData(null);
    setError(null);

    fetch(url)
        .then((response) => response.json())
        .then((data) => {

            // error handling for nonexistent data
            setIsLoading(false);
            if(data.cod >= 400) {
                setError(data.message);
                return;
            }
            setData(data);
        })
        .catch((error) => {
            setIsLoading(false);
            setError(error);
        });
  }, [url]);

  return { data, error, isLoading, setUrl };
};

export default UseFetch;

```


If we put sth other than a city, we get this message.


![error.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1605337796814/iNSR3GkzS.png)

## Wrapping Up

With this last touch to our app, we have completed all we need for this project. I hope you found it useful.

You can check out the [React Official Page](https://reactjs.org/docs/hooks-reference.html#gatsby-focus-wrapper) for a detailed explanation of the hooks. Also, you can deep dive into useEffect hook with this blog post from [Dan Abramov](https://overreacted.io/a-complete-guide-to-useeffect/)

You can find the source code [here](https://github.com/hulyak/weather-tutorial).

Thank you for reading and I hope you tried it yourself, too. This is my first experience as a writer, and it's actually really hard to follow where I was with the code and the tutorial. I hope I will get better soon 😃.

Also, feel free to connect with me on [Twitter](https://twitter.com/hulyakarakayaa), and [Github](https://github.com/hulyak).




