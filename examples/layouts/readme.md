# Layouts
A layout is just a component. You use a layout when working with nested routes. The core feature of a layout is that is outputs `children`. Often a layout is just a `div` that outputs children. Confusingly, HTML/CSS page layouts typically belong in a view.

## The Smallest Layout

```js
import React, { PropTypes } from 'react'
const MyLayout = ({ children }) => (<div>{children}</div>)
MyLayout.propTypes = { children: PropTypes.node.isRequired }
export default MyLayout
```
