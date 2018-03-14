---
extends: _layouts.documentation
title: "Plugins"
description: "Configuring and authoring third-party plugins."
---

<h2 style="visibility: hidden; font-size: 0; margin: 0 0 -1rem 0;">Overview</h2>

At their simplest, plugins are just functions that register new styles for Tailwind to inject into the user's stylesheet. That means that to get started authoring your own plugin, all you need to do is add an anonymous function to the `plugins` list in your config file:

```js
// ...

module.exports = {
  // ...
  plugins: [

    function({ addUtilities, addComponents, e, prefix, config }) {
      // This function is your plugin
    },

  ],
  // ...
}
```

Plugin functions receive a single object argument that can be [destructured](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) into several helper functions:

- `addUtilities()`, for registering new utility styles
- `addComponents()`, for registering new component styles
- `e()`, for escaping strings meant to be used in class names
- `prefix()`, for manually applying the user's configured prefix to parts of a selector
- `config()`, for looking up values in the user's Tailwind configuration

## Adding utilities

The `addUtilities` function allows you to register new styles to be output (along with the built-in utilities) at the `@@tailwind utilities` directive.

Plugin utilities are output in the order they are registered, *after* built-in utilities, so if a plugin targets any of the same properties as a built-in utility, the plugin utility will take precedence.

To add new utilities from a plugin, call `addUtilities`, passing in your styles using [CSS-in-JS syntax](#css-in-js-syntax):

```js
function({ addUtilities }) {
  const newUtilities = {
    '.skew-10deg': {
      transform: 'skewY(-10deg)',
    },
    '.skew-15deg': {
      transform: 'skewY(-15deg)',
    },
  }

  addUtilities(newUtilities)
}
```

### Prefix and important preferences

By default, plugin utilities automatically respect the user's [`prefix`](http://tailwind-docs.test/docs/configuration#prefix) and [`important`](http://tailwind-docs.test/docs/configuration#important) preferences.

That means that given this Tailwind configuration:

```js
// ...

module.exports = {
  // ...

  options: {
    prefix: 'tw-',
    important: true,
  },

}
```

...the example plugin above would generate the following CSS:

```css
.tw-skew-10deg {
  transform: skewY(-10deg) !important;
}
.tw-skew-15deg {
  transform: skewY(-15deg) !important;
}
```

If necessary, you can opt out of this behavior by passing an options object as a second parameter to `addUtilities`:

```js
function({ addUtilities }) {
  // ...

  addUtilities(newUtilities, {
    respectPrefix: false,
    respectImportant: false,  
  })
}
```

### Responsive and state variants

To generate responsive, hover, focus, active, or group-hover variants of your styles, specify the variants you'd like to generate using the `variants` option:

```js
function({ addUtilities }) {
  // ...

  addUtilities(newUtilities, {
    variants: ['responsive', 'hover'],
  })
}
```

If you only need to specify variants and don't need to opt-out of the default prefix or important options, you can also pass the array of variants as the second parameter directly:

```js
function({ addUtilities }) {
  // ...

  addUtilities(newUtilities, ['responsive', 'hover'])
}
```

## Adding components

The `addComponents` function allows you to register new styles to be output at the `@@tailwind components` directive.

Use it to add more opinionated, complex classes like buttons, form controls, alerts, etc; the sort of pre-built components you often see in other frameworks that you might need to override with utility classes.

To add new component styles from a plugin, call `addComponents`, passing in your styles using [CSS-in-JS syntax](#css-in-js-syntax):

```js
function({ addComponents }) {
  const buttons = {
    '.btn': {
      padding: '.5rem 1rem',
      borderRadius: '.25rem',
      fontWeight: '600',
    },
    '.btn-blue': {
      backgroundColor: '#3490dc',
      color: '#fff',
      '&:hover': {
        backgroundColor: '#2779bd'
      },
    },
    '.btn-red': {
      backgroundColor: '#e3342f',
      color: '#fff',
      '&:hover': {
        backgroundColor: '#cc1f1a'
      },
    },
  }

  addComponents(buttons)
}
```

### Prefix and important preferences

By default, component classes automatically respect the user's `prefix` preference, but **they are not affected** by the user's `important` preference.

That means that given this Tailwind configuration:

```js
// ...

module.exports = {
  // ...

  options: {
    prefix: 'tw-',
    important: true,
  },

}
```

...the example plugin above would generate the following CSS:

```css
.tw-btn {
  padding: .5rem 1rem;
  border-radius: .25rem;
  font-weight: 600;
}
.tw-btn-blue {
  background-color: #3490dc;
  color: #fff;
}
.tw-btn-blue:hover {
  background-color: #2779bd;
}
.tw-btn-blue {
  background-color: #e3342f;
  color: #fff;
}
.tw-btn-blue:hover {
  background-color: #cc1f1a;
}
```

Although there's rarely a good reason to make component declarations important, if you really need to do it you can always add `!important` manually:

```js
function({ addComponents }) {
  addComponents({
    '.btn': {
      padding: '.5rem 1rem !important',
      borderRadius: '.25rem !important',
      fontWeight: '600 !important',
    },
    // ...
  })
}
```

All classes in a selector will be prefixed, so if you add a more complex style like:

```js
function({ addComponents }) {
  addComponents({
    // ...
    '.navbar-inverse a.nav-link': {
        color: '#fff',
    }
  })
}
```

...the following CSS would be generated:

```css
.tw-navbar-inverse a.tw-nav-link {
    color: #fff;
}
```

To opt out of prefixing, pass an options object as a second parameter to `addComponents`:

```js
function({ addComponents }) {
  // ...

  addComponents(buttons, {
    respectPrefix: false,
  })
}
```

### Responsive and state variants

The `addComponents` function doesn't provide the ability to automatically generate variants since it doesn't typically make sense to do so for component classes.

You can always do this manually if necessary by wrapping your styles in the `@@variants` at-rule:

```js
function({ addComponents }) {
  addComponents({
    '@@variants responsive, hover': {
      '.btn': {
        padding: '.5rem 1rem !important',
        borderRadius: '.25rem !important',
        fontWeight: '600 !important',
      },
      // ...
    }
  })
}
```


## Escaping class names

## Manually prefixing

## Referencing the user's config

## Exposing options

## CSS-in-JS syntax

- Supports nested rules and media queries

## Using raw PostCSS
