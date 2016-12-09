# Standard Component
This is a standard component that relies on some props.

```js
import React, { PropTypes } from 'react'
import './SomethingClicky.scss'

export const SomethingClicky = ({ something, onClick }) => {
  return (
    <div className='SomethingClicky' onClick={onClick}>
      {something}
    </div>
  )
}

SomethingClicky.propTypes = {
  something: PropTypes.string.isRequired,
  onClick: PropTypes.func.isRequired
}

export default SomethingClicky

```

### Standard Usage
This is a component that includes another one and doesn't need any props.

```js
import React, { PropTypes } from 'react'
import SomethingClicky from './SomethingClicky'
import './SecondThing.scss'

export const SecondThing = () => {
  return (
    <div className='SecondThing'>
      <SomethingClicky something='Some text' onClick={(e) => console.log('clicked')}/>
    </div>
  )
}

export default SecondThing

```

## With Children
If you need to accept children.

```js
import React, { PropTypes } from 'react'
import './SomethingClicky.scss'

export const SomethingClicky = ({ children, onClick }) => {
  return (
    <div className='SomethingClicky' onClick={onClick}>
      {children}
    </div>
  )
}

SomethingClicky.propTypes = {
  children: PropTypes.node.isRequired, // <-- children don't *have* to be required
  onClick: PropTypes.func.isRequired
}

export default SomethingClicky

```
### Usage with children
This is a component that includes another one and doesn't need any props.

```js
import React, { PropTypes } from 'react'
import SomethingClicky from './SomethingClicky'
import './SecondThing.scss'

export const SecondThing = () => {
  return (
    <div className='SecondThing'>
      <SomethingClicky onClick={(e) => console.log('clicked')}>
        Some Text
      </SomethingClicky>
    </div>
  )
}

export default SecondThing

```

## Simple Component
If your component doesn't do anything interesting with props, you can shorten your component a little bit.

```js
import React, { PropTypes } from 'react'
import './SomethingClicky.scss'

export const SomethingClicky = ({ something, onClick }) => (
  <div className='SomethingClicky' onClick={onClick}>
    {something}
  </div>
)

SomethingClicky.propTypes = {
  something: PropTypes.string.isRequired,
  onClick: PropTypes.func.isRequired
}

export default SomethingClicky

```
