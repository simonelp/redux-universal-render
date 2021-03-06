## Redux Universal render

[![Build Status](https://travis-ci.org/StefanoPastore/redux-universal-render.svg?branch=master)](https://travis-ci.org/StefanoPastore/redux-universal-render) [![Coverage Status](https://coveralls.io/repos/github/StefanoPastore/redux-universal-render/badge.svg?branch=master)](https://coveralls.io/github/StefanoPastore/redux-universal-render?branch=master)

### Dependencies
[redux-thunk](https://github.com/gaearon/redux-thunk)

### Why you need?
It is necessary because you must wait data is loaded before render page on server side.

### Installation

To install the stable version with npm:

```
npm install --save redux-universal-render
```

### Usage
This module supply for you reducer and render function to apply on server and client.

Example store config:

```js
import { createStore, compose, applyMiddleware, combineReducers } from 'redux';
import { reducer as reduxUniversalRenderReducer } from 'redux-universal-render';
import thunk from 'redux-thunk';
import * as reducers from './reducers';

const initialState = {};

const store = createStore(
  combineReducers({ ...reducers, reduxUniversalRenderReducer }),
  initialState,
  compose(applyMiddleware(thunk))
);
```

Example server side rendering:

```js
import { awaitRender } from 'redux-universal-render';

match({ routes, location: req.originalUrl }, (error, redirectLocation, renderProps) => {
    const render = <Provider store={store}>
      <RouterContext {...renderProps} />
    </Provider>;
    awaitRender({
      store,
      render,
      cb: () => {
        const content = renderToString(render);
        const page = renderToStaticMarkup(<HTML content={content} />);

        res.send(`<!doctype html>\n${page}`);
      }
    }
    );
  });
});
```

Example actions creators:

```js
import { isParsed, addAction, parsedAction } from 'redux-universal-render';

export const syncActionCreator = () => ({ type: 'ACTION' });

const asyncName = 'asyncName';
export const asyncAction = () => (dispatch, getState) => {
  // not refetch data if it is already parsed for ssr and client render
  if (isParsed(asyncName)(getState())) return;

  dispatch(addAction(asyncName));
  setTimeout(() => {
    dispatch(syncActionCreator());
    dispatch(parsedAction(asyncName));
  }, 200);
};
```

### actions

  - addAction: This action add to pending async actions with unique name.

  Example:
  ```js
  import { addAction } from 'redux-universal-render';

  const name = 'action';
  export const action = () => (dispatch) => {
    dispatch(addAction(name));
  }
  ```

  - parsedAction: This action set ended a specific async action by same name adedd with `addAction`.

  Example:
  ```js
  import { addAction, parsedAction } from 'redux-universal-render';

  const name = 'action';
  export const action = () => async (dispatch) => {
    dispatch(addAction(name));
    // your async logic
    dispatch(parsedAction(name));
  }
  ```

  - errorAction: This action set in error a specific async action by same name adedd with `addAction`.

  Example:
  ```js
  import { addAction, errorAction } from 'redux-universal-render';

  const name = 'action';
  export const action = () => async (dispatch) => {
    dispatch(addAction(name));
    try {
      // your async logic
    } catch(e) {
      dispatch(errorAction(name, e));
    }
  }
  ```

### selectors

  - inExecution: return true if there is async actions in queue

  - isEnded: return true if there isn't async actions in queue

  - actions: return array of actions in queue

  - parsed: return array of actions already parsed

  - errors: return object with action name: error

  - isPending: return true if action name is added in queue

  - isParsed: return true if action name is already parsed

  - isError: return true if action name is in error


### API

  - awaitRender({

    * store (required): Redux store
    * render component (required): React component to render
    * cb (required): Function, it is called when all async actions dispatched are ended
    * ownerRender(walk) (optional): Function, receive function that render component you must call it can you wrap with your own business logic

  })

  - createActions(

    * name: string or function that receive getState and return unique name for actions
    * action: async action to await

  )

  Example:
  ```js
  import { createActions } from 'redux-universal-render';

  createActions((getState) => `name_${getState().myVar}`, async (dispatch, getState) => {
    // ... my logic
  });
  ```
