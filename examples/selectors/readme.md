# Selectors
Selectors take 1 argument, the state, and return a value from the state. If you need to pass additional arguments, use a curried function.

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

// a simple selector just grabs a key
export const selectPages = state => state.pages

// if you need additional arguments, use a curried selector
export const selectPageById = state => id => {
  const pages = selectPages(state)
  return pages && Array.isArray(pages) && pages.find(page => page.id === id)
}

// if your selector works with *part* of the state, name it appropriately for clarity
export const selectTitle = page => page && page.attributes && page.attributes.title
export const selectIsLoaded = page => page && page.meta && page.meta.isLoaded

export const selectPageTitleById = state => id => {
  const page = selectPageById(state)(id)
  return selectTitle(page)
}
```
