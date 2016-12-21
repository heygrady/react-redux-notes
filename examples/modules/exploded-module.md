# Exploded module
If your module has any complexity at all it's probably better to use an exploded module.

```md
- modules/myModule
  - actions/
  - constants/
  - middleware/
  - reducers/
    - index.js <-- should combine all reducers
    - thingsReducer.js <-- reducer that deals with `state.myModule.things`
  - selectors/
  - index.js <-- should return a reducer by default
```
