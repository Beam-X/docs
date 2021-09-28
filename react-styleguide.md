# React/JSX Style Guide

A mostly reasonable approach to React and JSX.

This style guide is mostly based on the standards that are currently prevalent in JavaScript, although some conventions (i.e async/await or static class fields) may still be included or prohibited on a case-by-case basis.

## Table of Contents

1. [Basic Rules](#basic-rules)
2. [Class vs React.createClass vs stateless](#class-vs-reactcreateclass-vs-stateless)
3. [Mixins](#mixins)
4. [Naming](#naming)
5. [Declaration](#declaration)
6. [Alignment](#alignment)
7. [Quotes](#quotes)
8. [Spacing](#spacing)
9. [Props](#props)
10. [Refs](#refs)
11. [Parentheses](#parentheses)
12. [Conditional Rendering](#conditional_rendering)
13. [Final Commas](#final_commas)
14. [Tags](#tags)
15. [Methods](#methods)
16. [Ordering](#ordering)
17. [isMounted](#ismounted)
18. [Legacy](#legacy)

## Basic Rules

Keep It Simple.

Only include one React component per file.
However, multiple [Stateless, or Pure, Components](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions) are allowed per file. eslint: [react/no-multi-comp](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless).

Always use classes over hooks.

Always use functional or PureComponent for simple components.
> When to use PureComponents? Component State/Props is an immutable object; A state don't have a multi-level nested object; Props don't have multi-level nested object.

Always use arrow functions.

Use PascalCase for components naming.

Use camelCase for const, classes, functions, etc. naming.

Do not use `React.createElement`.

Use ternary operators for conditional rendering.
```javascript
{isAuthorized ? <button>Logout</button> : null}
```

## Class vs React.createClass vs stateless

If you have internal state and/or refs, prefer `class extends React.Component` over `React.createClass`. eslint: [react/prefer-es6-class](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-es6-class.md) [react/prefer-stateless-function](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-stateless-function.md)

```javascript
// bad
const Listing = React.createClass({
  // ...
  render() {
    return <div>{this.state.hello}</div>;
  }
});

// good
class Listing extends React.Component {
  // ...
  render() {
    return <div>{this.state.hello}</div>;
  }
}
```

## Mixins

[Don't use mixins](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html)

> Why? Mixins introduce implicit dependencies, cause name clashes, and cause snowballing complexity. Most use cases for mixins can be accomplished in better ways via components, higher-order components, or utility modules.

## Naming

- **Extensions:** Use `.jsx` extension for React components
- **Filename:** Use PascalCase for filenames (e.g., `ReservationCard.jsx`)
- **Reference Naming:** Use PascalCase for React components and camelCase for their instances. eslint: [react/jsx-pascal-case](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

  ```javascript
  // bad
  import reservationCard from './ReservationCard';

  // good
  import ReservationCard from './ReservationCard';

  // bad
  const ReservationItem = <ReservationCard />;

  // good
  const reservationItem = <ReservationCard />;
  ```

- **Component Naming:** Use the filename as the component name. For example, `ReservationCard.jsx` should have a reference name of `ReservationCard`. However, for root components of a directory, use `index.jsx` as the filename and use the directory name as the component name:

  ```javascript
  // bad
  import Footer from './Footer/Footer';

  // bad
  import Footer from './Footer/index';

  // good
  import Footer from './Footer';

  ```

- **Props Naming:** Avoid using DOM component prop names for different purposes.
  
  > Why? People expect props like `style` and `className` to mean one specific thing. Varying this API for a subset of your app makes the code less readable and less maintainable, and may cause bugs.

  ```javascript
  // bad
  <MyComponent style='fancy' />

  // bad
  <MyComponent className='fancy' />

  // good
  <MyComponent variant='fancy' />
  ```

## Declaration

Do not use `displayName` for naming components. Instead, name the component by reference.

```javascript
// bad
export default React.createClass({
  displayName: 'ReservationCard',
  // stuff goes here
});

// good
class ReservationCard extends React.Component {
}
export default ReservationCard;

```

## Alignment

Follow these alignment styles for JSX syntax. eslint: [react/jsx-closing-bracket-location](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md) [react/jsx-closing-tag-location](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-tag-location.md)

```javascript
// bad
<Foo superLongParam='bar'
     anotherSuperLongParam='baz' />

// good
<Foo
  superLongParam='bar'
  anotherSuperLongParam='baz'
/>

// if props fit in one line then keep it on the same line
<Foo bar='bar' />

// children get indented normally
<Foo
  superLongParam='bar'
  anotherSuperLongParam='baz'
>
  <Quux />
</Foo>
```

## Quotes

Always use single quotes (') both for JS and JSX

```javascript
// bad
<Foo bar='bar' />

// good
<Foo bar='bar' />
```

## Spacing

- Always include a single space in your self-closing tag. eslint: [no-multi-spaces](https://eslint.org/docs/rules/no-multi-spaces), [react/jsx-tag-spacing](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-tag-spacing.md)

  ```javascript
  // bad
  <Foo/>

  // very bad
  <Foo                 />

  // bad
  <Foo
   />

  // good
  <Foo />
  ```

- Do not pad JSX curly braces with spaces. eslint: [react/jsx-curly-spacing](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-curly-spacing.md)

  ```javascript
  // bad
  <Foo bar={ baz } />

  // good
  <Foo bar={baz} />
  ```

- Use spacing in `if..else` statements

  ```javascript
  // bad
  if(condition) {
  }

  // bad
  if(condition){
  }

  // good
  if (condition) {
  }
  ```

- Use spacing for methods
  
  ```javascript
  // bad
  onClick=(object)=>{
  }

  // good
  onClick = (object) => {
  }
  ```

## Props

- Always use camelCase for prop names.

  ```javascript
  // bad
  <Foo
    UserName='hello'
    phone_number={12345678}
  />

  // good
  <Foo
    userName='hello'
    phoneNumber={12345678}
  />
  ```

- Omit the value of the prop when it is explicitly true. eslint: [react/jsx-boolean-value](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-boolean-value.md)
  
  ```javascript
  // bad
  <Foo
    hidden={true}
  />

  // good
  <Foo
    hidden
  />

  // good
  <Foo hidden />
  ```

- Always include an `alt` prop on `<img>` tags. If the image is presentational, alt can be an empty string or the `<img>` must have `role='presentation'`. eslint: [jsx-a11y/alt-text](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/alt-text.md)

  ```javascript
  // bad
  <img src='hello.jpg' />

  // good
  <img src='hello.jpg' alt='Me waving hello' />

  // good
  <img src='hello.jpg' alt='' />

  // good
  <img src='hello.jpg' role='presentation' />
  ```

- Do not use words like “image”, “photo”, or “picture” in `<img>` `alt` props. eslint: [jsx-a11y/img-redundant-alt](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/img-redundant-alt.md)

  > Why? Screenreaders already announce img elements as images, so there is no need to include this information in the alt text.

  ```javascript
  // bad
  <img src='hello.jpg' alt='Picture of me waving hello' />

  // good
  <img src='hello.jpg' alt='Me waving hello' />
  ```

- Use only valid, non-abstract [ARIA roles](https://www.w3.org/TR/wai-aria/roles#role_definitions). eslint: [jsx-a11y/aria-role](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/aria-role.md)

  ```javascript
  // bad - not an ARIA role
  <div role='datepicker' />

  // bad - abstract ARIA role
  <div role='range' />

  // good
  <div role='button' />
  ```

- Avoid using an array index as `key` prop, prefer a unique ID. ([why?](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318))

  ```javascript
  // bad
  {todos.map((todo, index) =>
    <Todo
      {...todo}
      key={index}
    />
  )}

  // good
  {todos.map(todo => (
    <Todo
      {...todo}
      key={todo.id}
    />
  ))}
  ```

- Spreading objects with known, explicit props. This can be particularly useful when testing React components with Mocha’s beforeEach construct.

  ```javascript
  export default function Foo {
    const props = {
      text: '',
      isPublished: false
    }

    return (<div {...props} />);
  }
  ```

  *Notes for use:* Filter out unnecessary props when possible. Also, use prop-types-exact to help prevent bugs.

  ```javascript
  // bad
  render() {
    const { irrelevantProp, ...relevantProps  } = this.props;
    return <WrappedComponent {...this.props} />
  }

  // good
  render() {
    const { irrelevantProp, ...relevantProps  } = this.props;
    return <WrappedComponent {...relevantProps} />
  }
  ```

## Refs

Always use ref callbacks. eslint: [react/no-string-refs](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-string-refs.md)

```javascript
// bad
<Foo
  ref='myRef'
/>

// good
<Foo
  ref={(ref) => { this.myRef = ref; }}
/>
```

## Parentheses

Wrap JSX tags in parentheses when they span more than one line. eslint: [react/jsx-wrap-multilines](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-wrap-multilines.md)

```javascript
// bad
render() {
  return <MyComponent variant='long body' foo='bar'>
           <MyChild />
         </MyComponent>;
}

// good
render() {
  return (
    <MyComponent variant='long body' foo='bar'>
      <MyChild />
    </MyComponent>
  );
}

// good, when single line
render() {
  const body = <div>hello</div>;
  return <MyComponent>{body}</MyComponent>;
}
```

## Conditional Rendering

- Use ternary operators for conditional rendering.
  
  ```javascript
  // bad 
  {isAuthorized && <button>Logout</button>}

  // good
  {isAuthorized ? <button>Logout</button> : null}
  ```

- Avoid nested conditions

  ```javascript
  // bad
  if (conditionA) {
    if (conditionB) {
    
    } else {
    
    }
  } else {
  
  }

  // good
  if (conditionA) {

  } else if (conditionB) {

  } else if (conditionC) {

  } else {

  }
  ```

- Avoid complex logic in condition, use boolean const in conditions.
  If your logic in condition statement because too heavy then it's better to take it out to a boolean const

  <!-- TODO: Add a better example -->
  ```javascript
  // bad
  if (conditionA && conditionB > 50 && conditionC < conditionD) {
  }

  // good
  const conditionB1 = conditionB > 50
  const conditionC1 = conditionC < conditionD
  if (conditionA && conditionB1 && conditionC1) {

  }

  ```

## Final Commas

Always add final commas for object/array elements
> Why? TLDR; Less diff count, less breaks of missed comma when add new item

```javascript
// bad
[
  {
    one,
    two
  },
  {
    one,
    two
  }
]


// good
[
  {
    one,
    two,
  },
  {
    one,
    two,
  },
]
```

## Tags

- Always self-close tags that have no children. eslint: [react/self-closing-comp](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md)

  ```javascript
  // bad
  <Foo variant='stuff'></Foo>

  // good
  <Foo variant='stuff' />
  ```

- If your component has multi-line properties, close its tag on a new line. eslint: [react/jsx-closing-bracket-location](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

  ```javascript
  // bad
  <Foo
    bar='bar'
    baz='baz' />

  // good
  <Foo
    bar='bar'
    baz='baz'
  />
  ```

## Methods

- Always use arrow function

  ```javascrip
  // bad
  function ItemList(props) {
    return (
      <ul>
        {props.items.map((item, index) => (
          <Item
            key={item.key}
            onClick={() => doSomethingWith(item.name, index)}
          />
        ))}
      </ul>
    );
  }

  // good
  const ItemList = (props) => {
    return (
      <ul>
        {props.items.map((item, index) => (
          <Item
            key={item.key}
            onClick={() => doSomethingWith(item.name, index)}
          />
        ))}
      </ul>
    );
  }
  ```

  Use arrow functions instead of binds

  ```javascript
  // bad
  <div onClick={this.onClickDiv.bind(this)} />

  // good
  <div onClick={() => {this.onClickDiv}} />
  ```

- Avoiding Inline Function Definition in the Render Function [Performance Tip]

  > Why? As far as functions are objects in JavaScript the inline function will always fail the prop diff when React does a diff check. Also, an arrow function will create a new instance of the function on every render if it’s used in a JSX property. This will create a lot of work for the garbage collector.

  ```javascript
  // bad
  export default class ItemsList extends React.Component {
    state = {
      item: [],
      selectedItemId: null
    }
    render(){
      const { items } = this.state;
      return (
        items.map(item => <Item key={item.id} onClick={() => this.setState({selectedItemId: item.id})} />
      )
    }
  }

  // good
  export default class ItemsList extends React.Component {
    state = {
      item: [],
      selectedItemId: null
    }
    handleItemSelect = (id) => {
      this.setState({selectedItemId: id})
    }
    render(){
      const { items } = this.state;
      return (
        items.map(item => <Item key={item.id} onClick={this.handleItemSelect} />
      )
    }
  }
  ```

- Do not use underscore prefix for internal methods of a React component.
  
  > Why? Underscore prefixes are sometimes used as a convention in other languages to denote privacy. But, unlike those languages, there is no native support for privacy in JavaScript, everything is public. Regardless of your intentions, adding underscore prefixes to your properties does not actually make them private, and any property (underscore-prefixed or not) should be treated as being public. See issues [#1024](https://github.com/airbnb/javascript/issues/1024), and [#490](https://github.com/airbnb/javascript/issues/490) for a more in-depth discussion.

  ```javascript
  // bad
  React.createClass({
    _onClick() {
      // do stuff
    },

    // other stuff
  });

  // good
  class extends React.Component {
    const onClick = () => {
      // do stuff
    }

    // other stuff
  }
  ```

- Be sure to return a value in your render methods. eslint: [react/require-render-return](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/require-render-return.md)

  ```javascript
  // bad
  render() {
    (<div />);
  }

  // good
  render() {
    return (<div />);
  }
  ```

## Ordering

- Ordering for `class extends React.Component`:

  1. optional `static` methods
  2. `constructor`
  3. `componentDidMount`
  4. `shouldComponentUpdate`
  5. `componentDidUpdate`
  6. `componentWillUnmount`
  7. clickHandlers or eventHandlers like `onClick` or `onChange`
  8. getter methods for `render` like `getSelectReason` or `getFooter`
  9. optional render methods like `renderNavigation` or `renderProfilePicture`
  10. `render`

- How to define `propTypes`, `defaultProps`, `contextTypes`, etc…
  
  ```javascript
  import React from 'react';
  import PropTypes from 'prop-types';

  const propTypes = {
    id: PropTypes.number.isRequired,
    url: PropTypes.string.isRequired,
    text: PropTypes.string,
  };

  const defaultProps = {
    text: 'Hello World',
  };

  class Link extends React.Component {
    static methodsAreOk() {
      return true;
    }

    render() {
      return <a href={this.props.url} data-id={this.props.id}>{this.props.text}</a>;
    }
  }

  Link.propTypes = propTypes;
  Link.defaultProps = defaultProps;

  export default Link;
  ```

## `isMounted`

Do not use `isMounted`. eslint: [react/no-is-mounted](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-is-mounted.md)

> Why? [`isMounted` is an anti-pattern](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html), is not available when using ES6 classes, and is on its way to being officially deprecated.

## Legacy

Don't use legacy functions (`getChildContext`, `componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate`)

---

Credits: this guide is based on [Airbnb JavaScript Style Guide](https://airbnb.io/javascript/react/#basic-rules) + our own changes according to principles we use in our company