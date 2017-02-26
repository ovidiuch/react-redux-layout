# react-redux-layout
Unleash your creativity with dynamic layouts!

> This is an **experimental project.** It isn't battle-tested and has some known limitations (more below).

Best suited for unconventional layouts, when the DOM is treated more like a canvas than a document. Yay for games, presentations and other full-screen experiences. Nay for articles, forums, e-commerce. YMMV.

![Flatris sizes](flatris-resize.gif)

## What does this solve?

Relationships between DOM elements in a responsive layout can get annoyingly hard to define via high level CSS.

A classic example is [scaling the font size to parent width.](http://stackoverflow.com/questions/16056591/font-scaling-based-on-width-of-container#comment29460412_19814948) This **can** be done traditionally using viewport-percentage lengths. But what if:

1. We want to scale `border-width` instead, ensure the value is a multiplier or 2 and constrain it between min/max values?
2. The parent is a cell in a grid with a dynamic number of columns? Depending on run time data, the parent might be `50vw` (two columns) or `25vw` (four columns). Or maybe those columns are actually resizable panes.

Whether we need to **scale values using custom functions** (1) or **adapt layout to user data** (2), sooner or later media queries and classes will not be enough.

I say "high level CSS" because we can always go rogue and compute each value by hand. We can **derive our layout from the viewport size**, trickling CSS attributes down the component hierarchy. This is precisely what react-redux-layout is about!

## How does this work?

First and foremost, this is **not another CSS-in-JS alternative!** It is complementary to [existing tools](https://github.com/MicheleBertoli/css-in-js#features). It might be helpful to separate styles as:

1. **Static styles.** Nothing changes here. Continue to set these via plain imports, css-modules, glamor, styled-jsx, styled-components, etc.
2. **Dynamic styles.** While the values are computed via react-redux-layout, how to assign them to elements is errbody's bidness. `style` object literals might suffice, but we can also pass the style values to something like jsxstyle or as styled-components props.

Now that we got that out of the way, let's get to the fun part: **Piggybacking Redux!**

> Warning: React and Redux knowledge required 🤓

### 1. Register layout reducer and init layout listener

```js
import initLayout, { layoutReducer } from 'react-redux-layout';
import computeLayout from './layout';

const store = combineReducers({
  layout: layoutReducer
});

initLayout({ store, computeLayout });

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

### 2. Define layout computer (pure function)

```js
const COLS = 6;
const ROWS = 10;

export default ({ width, height }) => {
  const landscape = width > height && width >= 640;
  const contentWidth = landscape ? floor(width / 2) : width;
  const colWidth = floor(contentWidth / COLS);

  return {
    content: {
      width: contentWidth,
      height
    },
    col: {
      width: colWidth,
      height: floor(height / ROWS)
    }
  };
};
```

### 3. Colocate layout data with component styles

```jsx
import { connect } from 'react-redux';

const Col = ({ styles }) => (
  <div styles={styles.root}>
    I'm so tired of JS
  </div>
);

export default connect(({ layout }) => ({
  styles: {
    root: layout.col.width
  }
}))(Col);

```

## Performance

Dynamic styles are slower and more verbose than classes. It's up to each of us to decide if the benefits outweigh the cost. Remember, most CSS attributes are static and should continue to be applied using classes.

## SSR

**Known limitation:** While the layout can be computed without a DOM, it needs a viewport (width, height) to derive the values from. The server can provide an arbitrary viewport, but the initial DOM will not match the client viewport until the client-side JS recomputes the layout. Workarounds:

- [Illustrated Algorithms](https://illustrated-algorithms.now.sh/) keeps initial DOM transparent and fades it in only after client-side rendering. Not ideal, but the static DOM is still used (React only needs to apply the diff between the dynamic style values).
- Don't use dynamic styles in the landing page. [Flatris](https://skidding.github.io/flatris/) uses a preloader styled with regular percentage values.

## This is insane!

This setup is liberating once in place. It's also pretty crazy, so [let me know](https://twitter.com/skidding) what you think!
