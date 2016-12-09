# Lifecycle Component
This is a component that needs to access some of the [React lifecycle methods](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle).

```js
import React, { PropTypes, Component } from 'react'

class SecondThing extends Component {
  componentWillMount () {
    const { onMount } = this.props
    onMount()
  }

  componentWillUnmount () {
    const { onBeforeUnmount } = this.props
    onBeforeUnmount()
  }

  render () {
    const { name, children } = this.props

    return (
      <div className='SecondThing'>
        <h2>Name</h2>
        {children}
      </div>
    )
  }
}

SecondThing.propTypes = {
  name: PropTypes.string.isRequired,
  children: PropTypes.node
}

export default SecondThing
```
