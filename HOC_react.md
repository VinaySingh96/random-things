---

## ✅ What is a Higher-Order Component?

> A **Higher-Order Component (HOC)** is a **function** that takes a component and returns a **new component with added functionality**.

---

## 🎯 Goal: Add a `loading` spinner to any component

---

### 1️⃣ Basic Component: `UserProfile`

```js
// UserProfile.js
import React from 'react';

const UserProfile = ({ user }) => {
  return (
    <div>
      <h2>User Profile</h2>
      <p>Name: {user}</p>
    </div>
  );
};

export default UserProfile;
```

---

### 2️⃣ HOC: `withLoading`

```js
// withLoading.js
import React from 'react';

const withLoading = (WrappedComponent) => {
  return function EnhancedComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <h3>Loading...</h3>;
    }

    return <WrappedComponent {...props} />;
  };
};

export default withLoading;
```

---

### 3️⃣ Usage: Wrap `UserProfile` with `withLoading`

```js
// App.js
import React, { useState, useEffect } from 'react';
import UserProfile from './UserProfile';
import withLoading from './withLoading';

const UserProfileWithLoading = withLoading(UserProfile);

function App() {
  const [loading, setLoading] = useState(true);
  const [user, setUser] = useState('');

  useEffect(() => {
    setTimeout(() => {
      setUser('Vinay Singh');
      setLoading(false);
    }, 2000); // Simulate API delay
  }, []);

  return (
    <div>
      <UserProfileWithLoading isLoading={loading} user={user} />
    </div>
  );
}

export default App;
```

---

## ✅ Output:

* ⏳ First 2 seconds: shows `Loading...`
* ✅ After 2 seconds: shows user profile

---

### 💡 Real-world use cases of HOCs:

| Use Case        | Example                     |
| --------------- | --------------------------- |
| Loading state   | `withLoading`               |
| Auth protection | `withAuthProtection`        |
| Error handling  | `withErrorBoundary`         |
| Theming         | `withTheme`                 |
| Reuse logic     | `withTimer`, `withTracking` |

---
