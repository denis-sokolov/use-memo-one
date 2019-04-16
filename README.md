# useMemoOne

[`useMemo`](https://reactjs.org/docs/hooks-reference.html#usememo) and [`useCallback`](https://reactjs.org/docs/hooks-reference.html#usecallback) with a stable cache (semantic guarantee)

[![Build Status](https://travis-ci.org/alexreardon/use-memo-one.svg?branch=master)](https://travis-ci.org/alexreardon/use-memo-one)
[![npm](https://img.shields.io/npm/v/use-memo-one.svg)](https://www.npmjs.com/package/use-memo-one)
[![dependencies](https://david-dm.org/alexreardon/use-memo-one.svg)](https://david-dm.org/alexreardon/use-memo-one)
[![min](https://img.shields.io/bundlephobia/min/use-memo-one.svg)](https://bundlephobia.com/result?p=use-memo-one)
[![minzip](https://img.shields.io/bundlephobia/minzip/use-memo-one.svg)](https://bundlephobia.com/result?p=use-memo-one)

## Background

`useMemo` and `useCallback` cache the most recent result. However, this cache can be destroyed by `React` when it wants to:

> You may rely on useMemo as a performance optimization, **not as a semantic guarantee**. In the future, **React may choose to “forget” some previously memoized values** and recalculate them on next render, e.g. to free memory for offscreen components. Write your code so that it still works without useMemo — and then add it to optimize performance. [- React docs](https://reactjs.org/docs/hooks-reference.html#usememo)

`useMemoOne` and `useCallbackOne` are `concurrent mode` safe alternatives to `useMemo` and `useCallback` **that do provide semantic guarantee**. What this means is that you will always get the same reference for a memoized value so long as there is no input change.

Using `useMemoOne` and `useCallbackOne` will consume more memory than `useMemo` and `useCallback` in order to provide a stable cache. `React` can release the cache of `useMemo` and `useCallback`, but `useMemoOne` will not release the cache until it is garbage collected.

## Install

```bash
# npm
npm install use-memo-one --save
# yarn
yarn add use-memo-one
```

## Usage

```js
import { useMemoOne, useCallbackOne } from 'use-memo-one';

function App(props) {
  const { name, age } = props;
  const value = useMemoOne(() => ({hello: name}), [name]);
  const getAge = useCallbackOne(() => age, [age])

  // ...
}
```

## API

See [`useMemo`](https://reactjs.org/docs/hooks-reference.html#usememo) and [`useCallback`](https://reactjs.org/docs/hooks-reference.html#usecallback)

## Exports

You can use two styles of imports

```js
// Avoids clashing with useMemo and useCallback
import { useMemoOne, useCallbackOne } from 'use-memo-one';

// Alternative: a drop in replacement
// This style also plays very well with eslint-plugin-react-hooks
import { useMemo, useCallback } from 'use-memo-one';
```

## Linting

`useMemo` and `useCallback` have fantastic linting rules with auto fixing in the [`eslint-plugin-react-hooks`](TODO) package. In order to take advantage of these with `useMemoOne` and `useCallbackOne`, structure your import like this:

```js
import { useMemo, useCallback } from 'use-memo-one';
// Or your can alias it yourself
import { useMemoOne as useMemo, useCallbackOne as useCallback} from 'use-memo-one';

function App() {
  const [isActive] = useState(false);

  const onClick = useCallback(() => {
    console.log('isActive', isActive);

  // the input array will now be correctly checked by eslint-plugin-react-hooks
  }, [isActive])
}
```

⚠️ This will only work if you use *only* `useMemoOne` in a file and not also `useMemo`. Otherwise you would not be able to alias `useMemoOne` to `useMemo` as it would clash

```js
import {useMemo} from 'react';

// ❌ alias clashes with useMemo
import {useMemoOne as useMemo} from 'use-memo-one';
```

If you want to be safe and ensure that `eslint-plugin-react-hooks` will always apply, you can add this `eslint` rule:

## eslint rules

Here are some eslint rules you are welcome to use

```js
module.exports = {
'rules': {
  // ...other rules

  'no-restricted-imports': [
    'error', {
        // If you want to force an application to always use useMemoOne
        paths: [
          {
            name: 'react',
            importNames: ['useMemo', 'useCallback'],
            message:
              'useMemo and useCallback are subject to cache busting. Please use useMemoOne and useCallbackOne',
          },
          // If you want to force use of the aliased imports from useMemoOne
          {
            name: 'use-memo-one',
            importNames: ['useMemoOne', 'useCallbackOne'],
            message:
              'use-memo-one exports `useMemo` and `useCallback` which work nicer with eslint-plugin-react-hooks',
          },
        ],
      },
    ],
  }
};
```