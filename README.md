# 🧁

**Zero-runtime CSS-in-TypeScript for CSS developers.**

Write your style sheets in TypeScript (or JavaScript) with local scoping of class names and CSS Variables, then generate purely static CSS files at build time.

Basically, it’s [“CSS Modules](https://github.com/css-modules/css-modules)-in-TypeScript” but with scoped CSS Variables + more.

---

**🚧 &nbsp; Please note, this is an alpha release.**

---

🔥 &nbsp; All styles generated at build time — just like [Sass](https://sass-lang.com), [Less](http://lesscss.org), etc.

✨ &nbsp; Minimal abstraction over standard CSS. All CSS features are available.

🌳 &nbsp; Locally scoped class names — just like [CSS Modules.](https://github.com/css-modules/css-modules)

🛠 &nbsp; Locally scoped custom property names, i.e. [CSS Variables.](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

🎨 &nbsp; High-level theme system with support for simultaneous themes. No globals!

💪 &nbsp; Type-safe styles thanks to [CSSType.](https://github.com/frenic/csstype)

🏃‍♂️ &nbsp; Optional runtime version for development and testing.

---

**Write your styles in `.css.ts` files.**

```ts
// styles.css.ts

import { createTheme, style } from '@mattsjones/css-core';

export const [themeClass, themeVars] = createTheme({
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});

export const exampleStyle = style({
  backgroundColor: themeVars.color.brand,
  fontFamily: themeVars.font.body,
  color: 'white',
  padding: 10
});
```

> 💡 These `.css.ts` files will be evaluated at build time. None of the code in these files will be included in your final bundle. Think of it as using TypeScript as your preprocessor.

**Then consume them in your markup.**

```ts
// app.ts

import { themeClass, exampleStyle } from './styles.css.ts';

document.write(`
  <section class="${themeClass}">
    <h1 class="${exampleStyle}">Hello world!</h1>
  </section>
`);
```

---

- [Setup](#setup)
- [API](#api)
  - [style](#style)
  - [globalStyle](#globalstyle)
  - [createTheme](#createtheme)
  - [createThemeVars](#createthemevars)
  - [createGlobalTheme](#createglobaltheme)
- [Advanced API](#advanced-api)
  - [mapToStyles](#maptostyles)
  - [createVar](#createvar)
  - [assignVars](#assignvars)
  - [inlineTheme](#inlinetheme)
- [Utility functions](#utility-functions)
  - [calc](#calc)
- [Thanks](#thanks)

---

## Setup

1. Install the dependencies.

```bash
$ yarn add --dev @mattsjones/css-core @mattsjones/css-babel-plugin @mattsjones/css-webpack-plugin
```

2. Add the [Babel](https://babeljs.io) plugin.

```json
{
  "plugins": ["@mattsjones/css-babel-plugin"]
}
```

3. Add the [webpack](https://webpack.js.org) plugin.

```js
const {
  TreatPlugin
} = require('@mattsjones/css-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  plugins: [
    new TreatPlugin({
      outputLoaders: [MiniCssExtractPlugin.loader]
    })
  ]
};
```

---

## API

### style

Create individual style rules.

```ts
import { style } from '@mattsjones/css-core';

export const className = style({
  display: 'flex'
});
```

Simple psuedos, selectors, CSS Variables (custom properties), `@media`/`@supports` queries and `@keyframes` are all supported.

```ts
import { style } from '@mattsjones/css-core';

export const className = style({
  display: 'flex',
  vars: {
    '--global-variable': 'purple'
  },
  ':hover': {
    color: 'red'
  },
  selectors: {
    '&:nth-child(2)': {
      background: '#fafafa'
    }
  },
  '@media': {
    'screen and (min-width: 768px)': {
      padding: 10
    }
  },
  '@supports': {
    '(display: grid)': {
      display: 'grid'
    }
  },
  '@keyframes': {
    from: {
      transform: 'rotate(0deg)'
    },
    to: {
      transform: 'rotate(359deg)'
    }
  },
  animation: '@keyframes 1.5s linear'
});
```

### globalStyle

Creates globally scoped CSS.

```ts
import { globalStyle } from '@mattsjones/css-core';

globalStyle('html, body', {
  margin: 0
});
```

### createTheme

Creates a locally scoped theme class and a collection of scoped CSS Variables.

```ts
import { createTheme, style } from '@mattsjones/css-core';

export const [themeClass, themeVars] = createTheme({
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});
```

You can create theme variants by passing a variables object as the first argument to `createTheme`.

```ts
import { createTheme, style } from '@mattsjones/css-core';

export const [themeA, themeVars] = createTheme({
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});

export const themeB = createTheme(themeVars, {
  color: {
    brand: 'pink'
  },
  font: {
    body: 'comic sans ms'
  }
});
```

> 💡 All theme variants must provide a value for every variable or it’s a type error.

### createThemeVars

Creates a collection of CSS Variables without coupling them to a specific theme implementation.

This is particularly useful if you want to split your themes into different bundles. In this case, your themes would be defined in separate files, but we'll keep this example simple.

```ts
import {
  createThemeVars,
  createTheme
} from '@mattsjones/css-core';

export const themeVars = createThemeVars({
  color: {
    brand: null
  },
  font: {
    body: null
  }
});

export const themeA = createTheme(themeVars, {
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});

export const themeB = createTheme(themeVars, {
  color: {
    brand: 'pink'
  },
  font: {
    body: 'comic sans ms'
  }
});
```

> 💡 All theme variants must provide a value for every variable or it’s a type error.

### createGlobalTheme

Creates a globally scoped theme, but with locally scoped variable names.

```ts
import { createGlobalTheme } from '@mattsjones/css-core';

export const themeVars = createGlobalTheme(':root', {
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});
```

---

## Advanced API

### mapToStyles

Creates an object that maps style names to hashed class names. This is particularly useful for mapping to component props, e.g. `<div className={styles.padding[props.padding]}>`

```ts
import { mapToStyles } from '@mattsjones/css-core';

export const padding = mapToStyles({
  small: { padding: 4 },
  medium: { padding: 8 },
  large: { padding: 16 }
});
```

You can also provide a map function as the second argument.

```ts
import { mapToStyles } from '@mattsjones/css-core';

const spaceScale = {
  small: 4,
  medium: 8,
  large: 16
};

export const padding = mapToStyles(spaceScale, (space) => ({
  padding: space
}));
```

### createVar

Creates a single CSS Variable.

```ts
import { createVar, style } from '@mattsjones/css-core';

export const colorVar = createVar();

export const exampleStyle = style({
  color: colorVar
});
```

Scoped variables can be set via the `vars` property on style objects.

```ts
import { createVar, style } from '@mattsjones/css-core';
import { colorVar } from './vars.css.ts';

export const parentStyle = style({
  vars: {
    [colorVar]: 'blue'
  }
});
```

### assignVars

Allows you to set an entire collection of CSS Variables anywhere within a style block.

```ts
import { style, assignVars } from '@mattsjones/css-core';
import { themeVars } from './vars.css.ts';

export const exampleStyle = style({
  vars: assignVars(themeVars.space, {
    small: 4,
    medium: 8,
    large: 16
  }),
  '@media': {
    'screen and (min-width: 1024px)': {
      vars: assignVars(themeVars.space, {
        small: 8,
        medium: 16,
        large: 32
      })
    }
  }
});
```

> 💡 All variables passed into this function must be assigned or it’s a type error.

### inlineTheme

Generates a custom theme at runtime as an inline style object.

```ts
import { inlineTheme } from '@mattsjones/css-core';
import { themeVars, exampleStyle } from './styles.css.ts';

const customTheme = inlineTheme(themeVars, {
  small: 4,
  medium: 8,
  large: 16
});

document.write(`
  <section style="${customTheme}">
    <h1 class="${exampleStyle}">Hello world!</h1>
  </section>
`);
```

## Utility functions

We also provide a standalone package of optional utility functions to make it easier to work with CSS in TypeScript.

> 💡 This package can be used with any CSS-in-JS library.

```bash
$ yarn add --dev @mattsjones/css-utils
```

### calc

Streamlines the creation of CSS calc expressions.

```ts
import { calc } from '@mattsjones/css-utils';

const styles = {
  height: calc.multiply('var(--grid-unit)', 2)
};
```

The following functions are available.

- `calc.add`
- `calc.subtract`
- `calc.multiply`
- `calc.divide`
- `calc.negate`

The `calc` export is also a function, providing a chainable API for complex calc expressions.

```ts
import { calc } from '@mattsjones/css-utils';

const styles = {
  marginTop: calc('var(--space-large)')
    .divide(2)
    .negate()
    .toString()
};
```

---

## Thanks

- [Nathan Nam Tran](https://twitter.com/naistran) for creating [css-in-js-loader](https://github.com/naistran/css-in-js-loader), which served as the initial starting point for [treat](https://seek-oss.github.io/treat), the precursor to this library.
- [Stitches](https://stitches.dev/) for getting us excited about CSS-Variables-in-JS.
- [SEEK](https://www.seek.com.au) for giving us the space to do interesting work.
