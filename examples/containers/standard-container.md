## Standard container

```js
import { connect } from 'react-redux'
import Something from '../components/SomeThing'

const mapStateToProps = (state, ownProps) => {
  return {
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
  }
}

const SomeThingContainer = connect(mapStateToProps, mapDispatchToProps)(SomeThing)
export default SomeThingContainer

```
