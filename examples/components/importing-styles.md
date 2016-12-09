# Standard Importing Styles
This is a standard component that relies on some props.

```js
import React, { PropTypes } from 'react'
import './SomethingClicky.scss' // <-- import it without assigning it to a variable

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

### Some Styles
The styles here will get applied globally, so be careful and consider [using BEM](http://getbem.com/naming/) to avoid class name conflicts.

```scss
.SomethingClicky {
  background-color: rgba(deeppink, 0.5);
}
```
