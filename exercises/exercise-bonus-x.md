# (BONUS) Exercise 6 - Convert Signals to Observables

## Use Observables instead of Signals to make life easier

```ts
const stat4e = signal({
  loading: false,
  data: null,
  error: null,
});

const getUsers = () => {
  state.set({
    loading: true,
    data: null,
    error: null,
  });

  fetch("https://example.com/api/users")
    .catch((errorMessage) => {
      state.set({
        loading: false,
        data: null,
        error: errorMessage,
      });
    })
    .then((response) => response.json())
    .then((users) => {
      state.set({
        loading: false,
        data: users,
        error: null,
      });
    });
};
```
