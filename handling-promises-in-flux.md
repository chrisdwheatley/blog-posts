Having recently started using [React](https://facebook.github.io/react/) & [Flux](https://facebook.github.io/flux/) one of the major pain points I came across was trying find a clean solution to combine the asynchronosity of promises with Flux's synchronous ecosystem.

I wanted to share the solution I ended up with. A solution which doesn't rely on a specific implementation of Flux and uses some new JavaScript language features & API's to help keep a clean, compact codebase.

**_A quick note before getting started_**

The code example below take advantages of ES6 features such as [modules](http://www.2ality.com/2014/09/es6-modules-final.html), [consts](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) and [arrow functions](https://github.com/esnext/es6-arrow-function). To write and run similar code you'll need to use a transpiler like [Babel](http://babeljs.io/) or [Traceur](https://github.com/google/traceur-compiler).

If you'd prefer to dive straight into the code you can check out the repository on [GitHub](https://github.com/swirlycheetah/react-flux-promises).

### The Problem

Let's say we want to display a list of every single GitHub emoji image and their relevant input code. GitHub conveniently provides an [API](https://developer.github.com/v3/emojis/) which lists all their emojis so we'll be fetching the data from there.

### The Solution

First off we'll create an entry point to our code. This file will be where React renders the component we'll be creating to the DOM.

##### _main.js_

```javascript
import React from 'react'
import EmojiComponent from './emojiComponent'

React.render(<EmojiComponent />, document.body)
```

Now let's create the `EmojiComponent` React component imported in `main.js`.

Until we've plugged in the data fetching code we'll simply render some static content within the `getInitialState` method to ensure everything is working as expected.

#####_emojiComponent.jsx_

```javascript
import React from 'react'

export default React.createClass({
  displayName: 'EmojiComponent',
  propTypes: {
    all: React.PropTypes.array
  },
  getInitialState () {
    return {
      all: [{
        binding: 'static-example',
        source: 'static-example'
      }]
    }
  },
  render () {
    const items = this.state.all.map(item => {
      return (
        <li>
          :{item.binding}:
          <img src={item.source}></img>
        </li>
      )
    })
    return (
      <div>
        <ul>
          {items}
        </ul>
      </div>
    )
  }
})
```

Now we've got a basic UI it's time to hook things up in Flux and get data into the component.

Let's start by setting up some constants which will be used across various parts of the codebase. Exporting `consts` means we can be sure the values won't change and will always be read only.

##### _consts.js_

```javascript
export const API_URL = 'https://api.github.com/emojis'
export const CHANGE_EVENT = 'change'
export const GET_DATA = 'GET_DATA'
export const GOT_DATA = 'GOT_DATA'
```

Next is an implementation of Flux's dispatcher.

##### _emojiDispatcher.js_

```javascript
import {Dispatcher} from 'flux'
import assign from 'object-assign'

export default assign(new Dispatcher(), {
  handleServerAction (action) {
    this.dispatch({
      source: 'server',
      action: action
    })
  },
  handleViewAction (action) {
    this.dispatch({
      source: 'view',
      action: action
    })
  }
})

```

Next let's build an action creator which will send actions to the dispatcher. The action creator will be triggered when `getData` is called.

##### _emojiActionCreator.js_

```javascript
import EmojiDispatcher from './emojiDispatcher'
import {fetchData} from './fetch'
import {API_URL, GET_DATA, GOT_DATA} from './consts'

export function getData () {
  EmojiDispatcher.handleViewAction({
    type: GET_DATA
  })

  fetchData(API_URL)
}

export function gotData (data) {
  data.then(resp => {
    EmojiDispatcher.handleServerAction({
      type: GOT_DATA,
      all: resp
    })
  })
}

```

As you can see above the `getData` function calls `fetchData` from the yet uncreated fetch module, let's create that module. It's a basic XHR call using a [polyfill](https://github.com/github/fetch) to replicate the upcoming `window.fetch` functionality. This is where our promise is created.

##### _fetch.js_

```javascript
import fetch from 'fetch'
import {gotData} from './emojiActionCreator'

export function fetchData (url) {
  return window.fetch(url)
    .then(response => {
      gotData(response.json())
    })
}
```

You can see that within the `then` of the promise we're calling `gotData` from the `emojiActionCreator` module we created earlier and passing through the data returned from the API response. If you look back to `emojiActionCreator.js` you'll see the `gotData` function resolves the promise and sends the response data off to the dispatcher to handle.

Finally we need to set up a store to handle data formatting and returning the action's response to the view component. We begin by setting `data` to null, once the `getEmojis` function is triggered from the view component the data is formatted as required and returned.

##### _emojiStore.js_

```javascript
import {EventEmitter} from 'events'
import assign from 'object-assign'
import EmojiDispatcher from './emojiDispatcher'
import {CHANGE_EVENT, GOT_DATA} from './consts'

let data = null

let EmojiStore = assign({}, EventEmitter.prototype, {
  emitChange () {
    this.emit(CHANGE_EVENT)
  },
  addChangeListener (callback) {
    this.on(CHANGE_EVENT, callback)
  },
  removeChangeListener (callback) {
    this.removeListener(CHANGE_EVENT, callback)
  },
  getEmojis () {
    return Object.keys(data).map(key => {
      return {
        'binding': key,
        'source': data[key]
      }
    })
  }
})

EmojiStore.dispatchToken = EmojiDispatcher.register(payload => {
  let action = payload.action

  switch (action.type) {
    case GOT_DATA:
      data = action.all
      EmojiStore.emitChange()
      break
  }
})

export default EmojiStore
```

Now that all the Flux architecture is in place we can update the view component to pull in the data we want to use. Remove the static data we originally returned in the `getInitialState` method and return an empty array, this will be replaced once the promise has resolved and the store has updated. `componentWillMount`, which is called just before the view component mounts, includes the `getEmojis` call to `emojiStore`.

##### _emojiComponent.jsx_

```javascript
import React from 'react'
import EmojiStore from './emojiStore'
import {getData} from './emojiActionCreator'

export default React.createClass({
  displayName: 'EmojiComponent',
  propTypes: {
    all: React.PropTypes.array
  },
  getInitialState () {
    return {
      all: []
    }
  },
  componentWillMount () {
    getData()
    EmojiStore.addChangeListener(this.onChange)
  },
  onChange () {
    this.setState({
      all: EmojiStore.getEmojis()
    })
  },
  render () {
    const items = this.state.all.map(item => {
      return (
        <li>:{item.binding}: <img src={item.source}></img></li>
      )
    })
    return (
      <div>
        <ul>
          {items}
        </ul>
      </div>
    )
  }
})
```

Putting that all together, transpiling it and serving it in the browser should give you a  list of all GitHub's emoji codes and their relevant images all while keeping a simple, clean, scalable integration of promises within the Flux architecture.

Once again if you want to check out the code in full head over to [GitHub](https://github.com/swirlycheetah/react-flux-promises). The full setup takes advantage of great tools such as [jspm](http://jspm.io/), [SystemJS](https://github.com/systemjs/systemjs), [Babel](http://babeljs.io/) & [Gulp](http://gulpjs.com/).
