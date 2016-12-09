# Components

There are two ways to create a component.

- wrapped in a folder
- side-by-side in the `components` folder

## Folder Component

```md
assets/ <-- for images, etc
components/
  SomethingClicky/
    SomethingClicky.js <-- put a component here
    styles.scss
    index.js <-- important for importing
  SecondThing/ <-- each component gets a folder
```

## Side-by-side Component

```md
assets/ <-- for images, etc
components/ <-- all components are side-by-side
  SomethingClicky.js
  SomethingClicky.scss
  SecondThing.js
  SecondThing.scss
```
