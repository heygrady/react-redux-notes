# Memoizing Selectors
If you are creating a selector that must *do some work* &mdash; like searching through an array to find the right value &mdash; it's important to memoize your selector for performance reasons.

- We use `createSelector` to memoize based on state. The `reselect` library clears the cache when the state changes.
- We use `memoize` to memoize based on random arguments. The `memoizee` library clears the cache when function args change.
- The "outer" func, created with `createSelector` wraps the "inner" func, created by `memoize`
- When the state changes, the inner func is thrown away
- When the state *doesn't* change, the memoized inner func is returned from cache, saving time.
- The inner func will return cached results based on `id`. Multiple requests for the same ID will return cached results until the state changes.
- Curried selectors allow for fine-grained memoization

```js
// example state
const state = {
  pages: [
    {
      type: 'page',
      id: 'page-id-1',
      attributes: { title: 'hello' },
      relationships: {
        author: { type: 'user', id: 'user-id-1' }
      },
      meta: { isLoaded: true }
    }
  ]
  users: [ { type: 'user', id: 'user-id-1', attributes: { name: 'Person Name' } } ]
}

// you will need reselect *and* memoizee
import { createSelector } from 'reselect'
import memoize from 'memoizee'

// simple selectors don't need memoized
export const selectPages = state => state.pages

// this is *not* memoized
export const selectPageById = state => id => {
  const pages = selectPages(state)
  return pages && Array.isArray(pages) && pages.find(page => page.id === id)
}

// here's the same selector, fully memoized
// @see https://github.com/reactjs/reselect
// @see https://github.com/medikoo/memoizee
export const selectPageBySlug = createSelector(
  selectPages, // <-- createSelector passes the result as the first arg to the func below
  // we return a function, making this a curried selector
  // we need to memoize the internal function as well
  pages => memoize(id => pages && Array.isArray(pages) && pages.find(page => page.id === id))
)

```
