# ramda-sheep
𝑓(🐏) = 🐑, A set of useful functions built on top of Ramda that are not in the core. 

Function are defined in a readme for now. 

If you are looking for more advanced set of extensions look at [ramda-adjunct](https://github.com/char0n/ramda-adjunct)

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
const isNilOrEmpty = R.either(R.isNil, R.isEmpty)
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
        (acc, itemsMap) =>
          R.update(itemsMap.index, itemsMap.item, acc),
        originalItems
      ),
      R.map(replacementItem => ({
        item: replacementItem,
        index: R.findIndex(
          originalItem => predicate(originalItem, replacementItem),
          originalItems
        )
      }))
    )(replaceWithItems))
```

usage:

```js
replaceBy((original, updated) => original.id === updated.id, updatedUsers, allUsers)
```

## pluckPath(strPath, list)

Like `R.pluck`, but with string path as a first argument. `R.pluck` allows to specify property name, while `pluckPath` can work with propert pathes like `myProp.otherProp.nestedProp`

```js
const pluckPath = R.pipe(
  R.split('.'),
  R.path,
  R.map,
)
const pluckPathCurr = R.curry((path, obj) => pluckPath(path)(obj))
```

usage:

```js
const list = [
  { x: { y: 1} },
  { x: { y: 2} },
  { x: { y: 3} },
]

pluckPath('x.y')(list) // [1, 2, 3]
```

## reduceObj

Like R.reduce, but for objects.

```js
const reduceObj = R.useWith(R.reduce, [R.identity, R.identity, R.toPairs]);
```

usage: 

```js
reduceObj((acc, [key, val]) => return acc, {}, obj)
```


## transformObjectDeep
Recursively traverse object and apply transformer function to every `[key, value]` pair and form new object based on that. If transformation function returns `undefined` then those `[key, value]` paris get removed from the original object. Keeps arrays as-is, meaning object inside an array won't be transformed.

```js
const reduceObj = R.useWith(R.reduce, [R.identity, R.identity, R.toPairs]);

// given key value pair return new key value pair or nothing
const removeEmptyKeysTransformer = ([key, val]) => {
  if (val === undefined) {
    return undefined;
  }
  return [key, val];
};

// transformer is a function that takes [key, value] and return transformed pair or undefined
// if undefined is returned that pair is going to be removed from the object. Recursively applied
// properties of the object. Keeps arrays as-is, meaning object inside an array won't be transformed.
const transformObjectDeep = R.curry((transformer, obj) => {
  return reduceObj(
    (acc, [key, val]) => {
      if (!Array.isArray(val) && typeof val === 'object') {
        const transformedVal = transformObjectDeep(transformer, val);

        if (R.keys(transformedVal).length === 0) {
          return acc;
        }
        acc[key] = transformedVal;
        return acc;
      }

      const transformedKeyVal = transformer([key, val]);

      if (transformedKeyVal === undefined) {
        return acc;
      }

      const [tKey, tVal] = transformedKeyVal;
      acc[tKey] = tVal;

      return acc;
    },
    {},
    obj,
  );
});
```

usage: 
```js
// given key value pair return new key value pair or nothing
const removeEmptyKeysTransformer = ([key, val]) => {
  if (val === undefined) {
    return undefined;
  }
  return [key, val];
};

transformObjectDeep(removeEmptyKeysTransformer, { x: 8, y: undefined } ); //output: { x: 8 }
```
