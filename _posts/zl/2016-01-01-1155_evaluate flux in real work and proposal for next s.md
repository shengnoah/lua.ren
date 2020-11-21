---
layout: post
title: evaluate flux in real work and proposal for next step 
tags: [lua文章]
categories: [topic]
---
I used Flux in recent work and after some evaluation, we decide to use it for
future development.

## 1\. Intro

### 1.1 React and Flux

[React](https://facebook.github.io/react/docs/tutorial.html) is a library to
build **view**. You can build abstract UI components in a declarative way,
start with basic ones and finally compose a complex UI. React component is
state machine, the whole UI is predictable according to the state.

[Flux](https://facebook.github.io/flux/docs/overview.html) is an application
architecture, which defines the way of organizing code and data flow. In
Facebook’s example implementation, _Store_ is the data, _View_ is the
components. _View_ will render based on _Store_ , and _Actions_ can be
triggered from _View_ to update _Store_.

![flux flow](!--￼1-->/images/flux-flow.jpg)

From _Store_ to _View_ to _Store_ , the data flows only in one direction. It
makes the application predictable. Every change in _Store_ will re-render the
whole app and it’s still fast thanks to virtual dom.

### 1.2 ES6

#### 1.2.1 Intro

The JavaScript programming language is standardized by ECMA (a standards body
like W3C) under the name ECMAScript. ES6 aka ECMAScript 6 or 2015, is a
ECMAScript standard approved in the summer of 2015.

#### 1.2.1 Compatibility

It is basically not supported by all browsers (2015.10), check the
[compatibility](https://kangax.github.io/compat-table/es6/).

#### 1.2.2 Why learn it?

  * It’s the trend, and most new JS frameworks use it.
  * react recommend writing components using ES6 class, and mixin will be removed and [Higher-Order Components] (<http://jamesknelson.com/structuring-react-applications-higher-order-components/>) should be used.

#### 1.2.3 How to use it?

We need to convert ES6 code to ES5 code. For example, use module bundler to
convert and bundle the JS files (`Webpack` \+ `babel-loader`).

I am working on a [cheatsheet for ES6] (/learnES6/).  
[ES6 features](https://github.com/lukehoban/es6features)

## 2 Evaluate flux in real work

### 2.1 Summary

 **Work** : `pivot viewer` is built using Flux, the implementation is based on
Facebook’s example.

**Conclusion** : using Flux can reduce bugs and make the development easier in
the following aspects:

  * modularize UI components. Each component’s interactions are restricted to itself, and external references should be explicitly referenced.
  * separation of logic. Components only care about what data they get (props) and what action will be triggered by which interaction. Updating data logic is in Store.
  * declarative way to build components. One direction data flow makes the interaction more clear.

Another benefit could be a better way to organize style sheet, to easily find
which style is applied to which component.

While on the other hand, **difficulties** :

  * many Flux implementations and no best practice yet. [Facebook itself has different implementations internally.] (<https://discuss.reactjs.org/t/lost-on-what-flux-implementation-to-use/545/12>).
  * learn: 
    * React 
    * JS (CommonJS format, ES6)
    * module bundler (`Browserify`, `Webpack`), organize JS and style sheet
    * new test API ()
  * building form-like widget which contains data update and re-render is complex (maybe solvable)
  * rewrite existing widgets (wrap kendo widget using react, etc)
  * some widgets that manipulate dom is not supported in react
  * the performance is bad when you need to re-render a lot components without reusing (e.g. table cells). (should be solvable)

### 2.2 Current status

The project is a Java Maven project, a multi-page application in JSP. JS is
written in plain IIFE style. CSS is not organized either.

For `pivot viewer`:

  * JS is written in ES5, CommonJS style (simply add `export` in JS file)
  * React 0.13, using `React.createClass` and mixin
  * `frontend-maven-plugin` will call `npm` during Maven build phase `generate-resources`, which calls `Browserify` to bundle the JS files
  * `wro4j` will reference the JS file in JSP file, it will also minify the file and append hash to the URL
  * styles for components are placed in different files, referenced in JSP.
  * no test cases

### 2.3 Next step

 **The first step** should be reorganizing the files and use `Webpack` to
bundle file, because:

  * we need to separate different entry point if there will be more pages
  * `Webpack` support multi entry points (each page has a entry point)
  * `Webpack` can support ES6 and other module standard 
  * `Webpack` can bundle style sheets
  * `Webpack` has some powerful features like `hot module reload` for development

**The second step** is to reorganize style sheets and use SASS (file extension
is **.scss** ) , because:

  * CSS files and class names are messy 
  * SASS is more powerful than CSS, especially for code reuse

One decision to be made: _inline or external CSS?_

  * inline: style sheet is inserted to source code in `style` tag. This means no caching in browser.
  * external CSS: one CSS file for one entry point, need to be referenced in JSP. This will add one more HTTP request.
  * In both ways, the style sheet is global scope. Still not perfect.

The above two steps has been validated to be working in my test, and it will
take one or two hour to apply and test it.

**Further steps:**

  * add test cases
  * rewrite components following ES6 and React 0.14
  * refactoring, since some of them are not React, some logic should not be in _Store_

**Further further steps** or in the mean time:

  * evaluate Redux, since it’s highly recommended, like [here] (<http://jamesknelson.com/which-flux-implementation-should-i-use-with-react/>) and [here] (<https://github.com/rackt/redux>)
  * evaluate [CSS modules] (<https://github.com/CSS-modules/CSS-modules>), which is modular, reusable, using no global scope; but a bit complex to use.
  * build a style guide
  * use `hot module reload` in development

### 2.4 Testing

#### 2.4.1 JSET

[JEST](https://facebook.github.io/jest/docs/tutorial-react.html#content) is a
unit test framework promoted by Facebook. It’s:

>   * Built on top of the Jasmine test framework, using familiar
> expect(value).toBe(other) assertions
>   * Automatically mocks CommonJS modules returned by require(), making most
> existing code testable
>   * DOM APIs are mocked and tests run in parallel via a small node.js
> command line utility
>

It can unit test single component and automatically mock others, but we have
some issues:

  * slow, a couple seconds for a test case in watch mode
  * Kendo UI is referenced in HTML globally, which can not be imported/required as a module. It’s OK when bundling the JS files and visit the web page, but in a unit test, kendo UI needs to be imported somehow. There is no support for npm package for Kendo UI.
  * no DEBUG in browser [yet] (<https://github.com/facebook/jest/issues/139>).

What’s more, it requires some learning for [`ReactTestUtils.Simulate`
API](https://facebook.github.io/react/docs/test-utils.html), which is used to
simulate events.

#### 2.4.2 Karma plus karma-webpack

The proposal is using `karma` with `karma-webpack`, which:

  * is faster than JEST
  * can add global reference like Kendo UI using `Karma`
  * DEBUG in browser

## 3 Plan

After some discussion, this is the final plan:

### 3.1 Folder structure

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    

|

    
    
    |-build            Output folder for bundled JS and CSS file   
    |---pivot.js  
    |---pivot.css  
    |-components       components can be reused (those who do not directly connected to Store)  
    |---kendo          kendo components rewritten  
    |-dispatcher       reference dispatcher by Facebook (do not touch)  
    |-lib              JS utilities (parse data, build pie chart using d3)  
    |-mixin            mixin  
    |-services         JS services, communicate with API, ajax calls  
    |-styles           global styles can be referenced by others  
    |---variables.scss define variables  
    |---helper.scss    define helper classes, like hide, float etc.  
    |---...              
    |-pages            folder contain all pages, each page has a entry JS file and a folder  
    |---pivot          single page folder, containing corresponding actions, components, constants and stores  
    |-----pivot.js     entry point for pivot page   
    |-----actions  
    |-----components  
    |-------List.jsx   a component named 'List'  
    |-------List.scss  style for 'List'    
    |-----constants  
    |-----stores  
      
  
---|---  
  
### 3.2 Work flow

Create a new page called `SuperViewer`:

  * create new folder `SuperViewer` and entry point `SuperViewer.js` under folder `pages`.
  * create components, actions, constants, stores, styles, etc.
  * for a component, reference required style sheet in JS file.
  * add entry point `SuperViewer.js` in `/app-web/webpack.config.js`
  * for development, execute `npm run dev` in terminal (`/app-web`), Webpack will watch the changes and automatically re-bundle.
  * for production, execute `npm run prod` in terminal, files will be bundled
  * for testing, execute `npm run test-flux` for simple testing (executed in build process). Execute `npm run test-flux-debug` for debugging.

  * [package.json] (../files/package.json)

  * [webpack.config.js] (../files/webpack.config.js)
  * [pom.xml] (../files/pom.xml)
  * [karma.flux.conf.js] (../files/karma.flux.conf.js)
  * [karma.flux.ci.conf.js] (../files/karma.flux.ci.conf.js)

### 3.3 TODO

  * upgrade to React 0.14
  * build style guide, naming conventions
  * rewrite components with ES6 syntax and style guide
  * refactoring code:
    * separate logic from store
    * make more component reusable
  * investigate building complex React component
    * form
    * large grid
  * write test cases
  * Redux seems nice

### 3.3 Style guide

This is some references, we need to build our own:

  * React/JSX: [Airbnb React/JSX Style Guide] (<https://github.com/airbnb/javascript/tree/master/react>)
  * CSS/SASS: [Airbnb CSS / Sass Styleguide] (<https://github.com/airbnb/CSS>)
  * CSS naming: [oocss and bem?] (<https://github.com/airbnb/CSS#oocss-and-bem>)