# ramda-sheep
𝑓(🐏) = 🐑, A set of useful functions built on top of Ramda that are not in the core. 

Function are defined in a readme for now.

## get
`String → {s: a} → a`

Just like lodash `get` accepts `.` delimeted path instead of an array:
```js
const pathGetter = R.compose(R.path, R.split('.'))
const get = R.curry((path, obj) => pathGetter(path)(obj))
```

usage:
```js
get('a.b', {a: {b: 8}}) //8
```


## set 
`String → a → {a} → {a}`

Sets a property on an object using given path, returns new object.
```js
const pathSetter = R.compose(R.assocPath, R.split('.'))
const set = R.curry((path, value, obj) => pathSetter(path)(value, obj))
```

usage:
```js
set('a.b.c', 8, {}) // {a: {b: {c: 8}}}
```

## isNilOrEmpty 
`Obj → Boolean`

Checks if value is nil or empty.

```js
const isNilOrEmpty = R.converge(R.or, [R.isNil, R.isEmpty])
```

## differenceAllWith(predicate, listOfLists)

Same as differenceWith, but for multiple lists
```js
  const differenceAllWith = R.curry((predicate, array) =>
    R.reduce(
      R.differenceWith(predicate),
      R.head(array),
      R.tail(array)
  ))
```

## replaceBy(predicate, replaceWithItems, originalItems) 
`((originalItem, itemToReplaceWith) → Boolean) → Array → Array → Array`

Replaces original items with given array of items using predicate, if predicate returns true, item will be replaced in original items in-place (it's orignal index) with the item from "replaceWithItems".

```js
  const replaceBy = R.curry((predicate, replaceWithItems, originalItems) =>
    R.compose(
      R.reduce(
        (eventsAcc, eventMap) =>
          R.update(eventMap.index, eventMap.event, eventsAcc),
        originalItems
      ),
      R.map(replacementItem => ({
        event: replacementItem,
        index: R.findIndex(
          targetItem => predicate(targetItem, replacementItem),
          originalItems
        )
      }))
    )(replaceWithItems))
```

usage:

```js
replaceBy((original, updated) => original.id === updated.id, updatedUsers, allUsers)
```
