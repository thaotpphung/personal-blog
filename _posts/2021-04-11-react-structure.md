---
layout: post
title: "Writing React and Redux Applications: A Detailed Look Into Structuring Projects"
date: 2021-04-11 09:04:23
description: Looking for the best ways to structure React and Redux applications.
tags: [javascript, code, structure]
---

There is no _One True Project Structure_ when it comes to Redux. With this in mind, I'll explain what has worked for me.

## General structure

There are two approaches for grouping files:

* By concept: one directory for components, reducers, actions, and so on.
* By page: all files related to one page are inside the same directory.

Initially I started off with grouping by concept.

Grouping by concept works for small applications, but in order to add new pages, you have to make note of many files: `reducers/MyPage.js`, `selectors/MyPage.js`, `components/MyPage.js` and a few more, depending on the technologies used. This makes scaling difficult, and training additional developers for the project can deem tricky.

Since that didn't work, I tried grouping files by view or page: `dashboard`, `users`, etc.

Here are main directories I have in my application:

* `app`: Redux store configuration and root reducer, React Router routes, application root components with connections to Redux, Redux DevTools, React Router, and other stack-specific tooling.
* `components`: shared components.
* `util`: generic functions.
* `pages`: features (could also be named `views`).

`pages` directories look like this:

```bash
pages/
  page-name/
    components/
      PageNameView.jsx
      PageNameLayout.jsx
      *.jsx
    duck.js
  very-big-page/
    page1/
    page2/
```

* I follow [the Ducks convention](https://github.com/erikras/ducks-modular-redux) so instead of separate files for actions, reducers and constants there are single `duck` file. I also put Reselect selectors into the duck file as `selector` named export.
* The `components` directory contains all React components that are unique to this view.
* `PageNameView` is connected to Redux and contains all action calls. Nested components receive handlers as props and don’t know anything about Redux. All page layout goes to `PageNameLayout` component.

## Components

For simple applications I like the simplest structure:

```bash
page-one/
  Component1.js
  Component2.js
  Component2.css
```

It works fine until you add more styles and other non-JavaScript files to your components. In this case I’d put every component into a separate directory:

```bash
page-one/
  Component1
    index.js
    Component1.js
    Component1.css
    Component1.spec.js
```

File names are still contain component class names so you can open them using a fuzzy search in your editor. I also have an extra entry file in every component directory, `index.js`:

```javascript
export { default } from './Component1';
```

It allows you to import components like this: `components/Component1` instead of `components/Component1/Component1`.

## Ducks and selectors

Each page has its own duck file `duck.js` structured as follows:

```javascript
import { combineReducers } from 'redux';
import { createStructuredSelector } from 'reselect';

const DO_SOMETHING_COOL = 'myapp/page-name/DO_SOMETHING_COOL';

// Actions

export function doSomethingCool(what) {
  return {
    type: DO_SOMETHING_COOL
    // ...
  };
}

// Reducers

function cookiesReducer(state, action) {
  switch (action.type) {
    case DO_SOMETHING_COOL:
      return {
        /*_*/
      };
    default:
      return {
        /*_*/
      };
  }
}

export default combineReducers({
  cookies: cookiesReducer
  // ...
});

// Selectors

const isFetching = state => state.isFetching;
const cookies = state => state.data.cookies;

export const selector = createStructuredSelector({
  isFetching,
  cookies
});
```

Then import it at the root reducer, `app/reducers.js`:

```javascript
import { combineReducers } from 'redux';
import pageName from '../pages/page-name/duck';

export default combineReducers({
  pageName
  // ...
});
```

I use selectors as the only way to access Redux state in components. So I connect selectors and actions to page’s root component, `PageNameView.jsx`:

```javascript
import React, { Component, PropTypes } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as duck from '../duck';
import PageNameLayout from './PageNameLayout';

@connect(
  state => duck.selector(state.pageName),
  dispatch => ({
    actions: bindActionCreators(duck, dispatch)
  })
)
export default class PageNameView extends Component {
  /*
  Here you have:
  - this.props.actions.doSomethingCool
  - this.props.isFetching
  - this.props.cookies
  */

  handleSomethingCool(what) {
    this.props.actions.doSomethingCool(what);
  }

  render() {
    const { isFetching } = this.props;
    return (
      <div>
        {isFetching ? (
          <span>Loading...</span>
        ) : (
          <PageNameLayout
            {...this.props}
            onSomethingCool={what => this.handleSomethingCool(what)}
          />
        )}
      </div>
    );
  }
}
```

## Development and production versions

In a few places I have separate files for development and production builds: `containers/Root` and `store/configureStore`, for example. I think they are easier to use then a single file with a bunch of `if`s.

For example, `configureStore.development.js` and `configureStore.production.js` are completely independent modules and `configureStore.js` chooses which one to use:

```javascript
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./configureStore.production');
} else {
  module.exports = require('./configureStore.development');
}
```

It allows you to do `import configureStore from './store/configureStore` and have the right version of the module depending on the current environment.

## Other approaches

* [Structuring React Projects](https://survivejs.com/webpack_react/structuring_react_projects/), a chapter from SurviveJS book.
* [Organizing Large React Applications](http://engineering.kapost.com/2016/01/organizing-large-react-applications/)
* [How to better organize your React applications](https://medium.com/@alexmngn/how-to-better-organize-your-react-applications-2fd3ea1920f1#.sbykc54ta)
