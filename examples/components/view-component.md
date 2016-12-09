# Standard View Component
A view component is nothing special -- it's just a component. A view is used in a route to render that route. Typically a view does not need to accept arguments but it may if the route passes params.

```js
import React from 'react'
import './MyPageView.scss'

import Header from './MyPageHeader'
import Sidebar from './MyPageSidebar'
import Main from './MyPageMain'

export const MyPageView = () => (
  <div className='MyPageView'>
    <Header />
    <Sidebar />
    <Main />
  </div>
)

export default MyPageView

```

## With Route Params
Sometimes you need to access the [injected props](https://github.com/ReactTraining/react-router/blob/master/docs/API.md#injected-props) that come from `react-router`.

- Read about [the location object](https://github.com/mjackson/history/blob/v2.x/docs/Location.md)

```js
import React, { PropTypes } from 'react'
import './MyPageView.scss'

import Header from './MyPageHeader'
import Sidebar from './MyPageSidebar'
import Main from './MyPageMain'

export const MyPageView = ({ params, location }) => {
  const { something } = params

  return (
    <div className='MyPageView'>
      <Header />
      <Sidebar />
      <Main>
        <h2>Viewing {something}</h2>
      </Main>
    </div>
  )
}
MyPageView.propTypes = {
  params: PropTypes.object.isRequired,
  location: PropTypes.object.isRequired
}

export default MyPageView

```
