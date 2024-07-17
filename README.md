# React + Redux + Redux Saga + TypeScript

## App.tsx

```js
import {
  applyMiddleware,
  combineReducers,
  legacy_createStore as createStore,
} from "redux";

import { useDispatch, useSelector } from "react-redux";
import createSagaMiddleware from "redux-saga";
import { call, put, takeEvery } from "redux-saga/effects";
import { User } from "./types";

// Create an Action
const GET_USERS_FETCH = "GET_USERS_FETCH";
const GET_USERS_SUCCESS = "GET_USERS_SUCCESS";

// Create an Action Creator
const getUsersFetch = () => ({ type: GET_USERS_FETCH });

// Create state
const initialState: { users: User[] } = {
  users: [],
};

// Create a Reducer
const userReducer = (
  state = initialState,
  action: { type: string; users: User[] }
) => {
  switch (action.type) {
    case GET_USERS_SUCCESS:
      return { ...state, users: action.users };
    default:
      return state;
  }
};

// Create a function to call the API
const usersFetch = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/users");
  const data = await response.json();
  return data as User[];
};

// Create a worker saga
function* workGetUsersFetch() {
  const users: User[] = yield call(usersFetch);
  yield put({ type: GET_USERS_SUCCESS, users });
}

// Create a watcher saga
function* mySaga() {
  yield takeEvery(GET_USERS_FETCH, workGetUsersFetch);
}

// redux & saga configurations
const sagaMiddleware = createSagaMiddleware();
const rootReducer = combineReducers({ users: userReducer });
export const store = createStore(
  rootReducer,
  {},
  applyMiddleware(sagaMiddleware)
);
sagaMiddleware.run(mySaga);
export type RootState = ReturnType<typeof store.getState>;

// UI Code
const App = () => {
  // User Redux dispatch & selector
  const dispatch = useDispatch();
  const users = useSelector((state: RootState) => state.users.users);
  return (
    <div>
      <button onClick={() => dispatch(getUsersFetch())}>Get Users</button>
      {users?.map((user: User) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  );
};

export default App;

```

## main.tsx

```js
import React from "react";
import ReactDOM from "react-dom/client";
import { Provider } from "react-redux";
import App, { store } from "./App.tsx";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);

```
