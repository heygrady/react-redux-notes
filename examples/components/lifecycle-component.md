# Lifecycle Component
This is a component that needs to access some of the [React lifecycle methods](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle).

```js
import React, { PropTypes, Component } from 'react'

class SecondThing extends Component {
  componentWillMount () {
    const { load } = this.props // <-- you can use props normally
    load() // <-- you'd use this to dispatch an action when the component loads
  }

  componentWillUnmount () { // <-- there are
    const { unload } = this.props
    unload()
  }

  render () {
    const { name, children } = this.props

    return (
      <div className='SecondThing'>
        <h2>{name}</h2>
        {children}
      </div>
    )
  }
}

SecondThing.propTypes = {
  name: PropTypes.string.isRequired,
  children: PropTypes.node,
  load: PropTypes.func.isRequired,
  unload: PropTypes.func.isRequired
}

export default SecondThing
```
