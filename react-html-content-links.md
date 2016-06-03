Working with the WordPress API you will inevitably run into a situation where the rendered HTML content contains links that you would like to be compatible with react-router. If you don't do anything at all, your internal links will cause the browser to reload instad of using react-router to transition the page. You might also want to do something special like tracking external links using this same method.

### Usage
```jsx
// defaults
<ContentWithLinks content={content} />

// custom match function, matching any href
<ContentWithLinks match={function (url) { return true }} content={content} />

// custom transform function
<ContentWithLinks transform={function (url) { return url + 'extra-stuff' }} content={content} />

// custom onClickLink function
<ContentWithLinks onClickLink={function (path) { console.log(path) }} content={content} />
```

### ContentWithLinks container
```jsx
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { push } from 'react-router-redux'

// convert the htmlString into an object that works with dangerouslySetInnerHTML
// @see https://facebook.github.io/react/tips/dangerously-set-inner-html.html
const createMarkup = (htmlString) => ({ __html: htmlString })

class ContentWithLinks extends Component {
  handleClick (e) {
    const { origin } = window.location

    // allow for href to be transformed if necessary
    const transform = this.props.transform || ((url) => url.substr(origin.length))

    // allow for a custom href match function
    const match = this.props.match || ((url) => !!url && url.startsWith(origin))

    // allow for custom onClickLink function
    const onClickLink = this.props.onClickLink || ((path) => {
      // @see http://stackoverflow.com/a/34863577/5149458
      const { dispatch } = this.props
      dispatch(push(path))
    })

    // grab the tagName and href
    let { tagName, href } = e.target

    if (tagName === 'A' && match(href)) {
      // stop the browser from following the link
      e.preventDefault()
      onClickLink(transform(href))
    }
  }

  componentDidMount () {
    if (this.element) {
      this.element.addEventListener('click', this.handleClick.bind(this))
    }
  }

  componentWillUnmount () {
    if (this.element) {
      this.element.removeEventListener('click', this.handleClick.bind(this))
    }
  }

  render () {
    const { content } = this.props
    const self = this

    return (
      <div dangerouslySetInnerHTML={createMarkup(content)} ref={function (ref) { self.element = ref }} />
    )
  }
}

ContentWithLinks.propTypes = {
  content: PropTypes.string.isRequired, // <-- expects a string containing raw html
  dispatch: PropTypes.func.isRequired, // <-- dispatch is injected by connect()
  match: PropTypes.func,
  onClickLink: PropTypes.func,
  transform: PropTypes.func
}

const Container = connect()(ContentWithLinks)
export default Container
```
