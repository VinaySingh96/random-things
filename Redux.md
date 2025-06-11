Sure! Here's a **minimal working example of Redux** in a React app using **Redux Toolkit**, which is the modern and recommended way to use Redux.

---

## ✅ Folder Structure:

```
src/
├── app/
│   └── store.js
├── features/
│   └── counter/
│       ├── counterSlice.js
│       └── Counter.js
├── App.js
└── index.js
```

---

## 1️⃣ `store.js` – Create Redux store

```js
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

---

## 2️⃣ `counterSlice.js` – Create slice

```js
// src/features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  value: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export default counterSlice.reducer;
```

---

## 3️⃣ `Counter.js` – UI Component

```js
// src/features/counter/Counter.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './counterSlice';

const Counter = () => {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>
        Increment by 5
      </button>
    </div>
  );
};

export default Counter;
```

---

## 4️⃣ `App.js`

```js
// src/App.js
import React from 'react';
import Counter from './features/counter/Counter';

function App() {
  return (
    <div className="App">
      <Counter />
    </div>
  );
}

export default App;
```

---

## 5️⃣ `index.js` – Wrap App in Provider

```js
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { Provider } from 'react-redux';
import { store } from './app/store';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

---

## ✅ To Install:

```bash
npm install @reduxjs/toolkit react-redux
```

---

### 🔁 Now your Redux setup includes:

* State: `counter.value`
* Actions: `increment`, `decrement`, `incrementByAmount`
* UI: Fully wired to Redux

Let me know if you want:

* Redux with async (`createAsyncThunk`)
* TypeScript version
* Persist Redux store with `localStorage` or `redux-persist`
