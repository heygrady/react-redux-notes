## Standard layout
A layout is used by react router for injecting index and child routes when using nested routes. Usually your layout simply needs to output children. In this case, `children` will always be the view for the index or child route.

```js
import React, { PropTypes } from 'react'

const MyLayout = ({ children }) => (<div>{children}</div>)

MyLayout.propTypes = {
  children: PropTypes.node.isRequired
}

export default MyLayout
```
