# Internal state component
This is a component that needs to manage state internally. Normally you want to keep state in Redux and that should still be the goal. In the button below the hover state is used to set a class name. The rest of the app does not care about that state so it's safe to use an internal state.

**Note:** Do not use internal state components except in rare cases where managing the state internally is more appropriate than storing it globally. It should pass the test of "only this instance of the component cares about the state."

```js
import React, { PropTypes, Component } from 'react'
import classnames from 'classnames'

class SomeButton extends Component {
  constructor () {
    super()
    this.hover = this.hover.bind(this) // <-- class functions need bound
    this.state = { isHovered: false } // <-- initialize state
  }

  hover (isHovered) {
    this.setState({ isHovered })
  }

  render () {
    const { label, toggle, isToggled } = this.props // <-- props still come from the outside
    const { isHovered } = this.state // <-- internal state comes in here

    const headerButtonClassName = classnames('SomeButton', {
      'SomeButton--toggled': isToggled, // <-- this comes from redux
      'SomeButton--hovered': isHovered
    })

    return (
      <button
        className={headerButtonClassName}
        onClick={toggle} // <-- this is a redux action
        onMouseEnter={() => this.hover(true)}
        onMouseLeave={() => this.hover(false)}
      >
        {label}
      </button>
    )
  }
}

SomeButton.propTypes = {
  label: PropTypes.string.isRequired,
  toggle: PropTypes.func,
  isToggled: PropTypes.bool
}

export default SomeButton

```
