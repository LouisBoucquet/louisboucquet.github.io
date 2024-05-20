# Null or Undefined what are they and which should you use?

In Javascript/Typescript there can be some confusion as to why both `null` and `undefined` exist.
They both encapsulate basically the same concept, a value that's not present.
In this blog post you will learn that there are important differences between `null` and `undefined`.

## Presence

Within Javascript, almost everything is an object which is basicaly a key-value map.
Because Javascript runs mostly on the web it tries to throw as few errors as possible, you don't want an entire site to fail because of one measly missed key.
So when trying to read a value from an object that is not present on the object, javascript defaults to returning `undefined`.

```ts
const emptyObject = {}
// Prints undefined
console.log(emptyObject.key);
```

This is a first weird kwerk of how `undefined` works.
When a key contains `undefined` you can't know wether it was just a value that wasn't present on the object you read or a value that was actualy present but with a value of `undefined`.

```ts
const obj = {}
// Prints undefined
console.log(obj.key)

const obj = {
	key: undefined
}
// Also prints undefined
console.log(obj.key)
```

When does this matter?
In both cases the value read from the object is `undefined`.
So in a sense an empty key is functionally the same as a value of `undefined`.
In a some cases though, there actually is a difference.

Here is an example of when this goes wrong.

```ts
function isComplete<T>(object: Partial<T>): object is T {
	return Object.values(object).every(value => value !== undefined);
}
```

This function is a type guard that checks if every key of a `Partial<...>` type is present.
Because absent keys won't be checked this code doesn't actually ensure a complete type.

```ts
type A = {
	a: string
}

// Correctly returns false
isComplete<A>({
	a: undefined
});

// Incorrectly returns true
isComplete<A>({});
```

<!-- To write this function properly you'd need to know which keys to check.
With this knowledge, let's improve the function:

```ts
function isComplete(object: object, keys: string[]): boolean {
	return keys
		.map(key => object[key])
		.every(value => value !== undefined);
}

// Returns false
isComplete({
	a: undefined
}, ['a']);

// Returns false
isComplete({}, ['a']);
```

Now the function correctly returns `false` when the key `a` is not present. -->

Another way to check for the presence of a key within an object is with the `in` keyword

```ts
// Evaluates to `true`
'key' in {key: 'value'}

// Evaluates to `true`
'key' in {key: undefined}

// Evaluates to `false`
'key' in {}
```

You have to be careful when using `in`, it doesn't guarantee that a value is not `undefined`.

### Typescript

In Typescript you can model your types to allow the absence of keys.

```ts
type Type = {
	key?: string;
}

// Ok
const t: Type = {
	key: 'value',
}

// Ok
const t: Type = {
}

// Not Ok
const t: Type = {
	key: undefined,
}
```

Typescript also allows you to require the presence of keys while still allowing the value to be `undefined`.

```ts
type Type = {
	key: string | undefined;
}

// Ok
const t: Type = {
	key: 'value',
}

// Ok
const t: Type = {
	key: undefined,
}

// Not Ok
const t: Type = {
}
```


### A Best practice

Dealing with the difference between absence and `undefined` can be annoying.
That's where `null` comes in, you can use the convention to only explicitly use `null`.
That way when you get `undefined` somewhere you don't have to wonder if it is a mistake or actually a vallue that has been set to `null`.

<!-- ## Difference from a JSON perspective

## Equality

When checking for both `null` and `undefined` at the same time you can use `value == null`.
Not using the strict equals (`===`) checks for both values. -->

## Final example

Imagine a function that takes in a config, using all the knowledge we've gathered let's implement this in a safe way.

```ts
type Config = {
	property: string | null
	// ...
}
```

Note that for the actual config we don't use absent (`key?: value`) or `undefined` (`key: value | undefined`) values.
This signifies that if `property` is `null` it has explicitly been set that way.

Now we want to write a function that accepts a `Config` value.
But we don't want the programmer to have to write out the whole config every time.
Instead, we allow the programmer to pass a `Partial<Config>` and we fill in the missing items.

```ts
declare const defaultConfig: Config;

function someFunction(config: Partial<Config>) {
	config = {
		...defaultConfig,
		...config,
	}
}
```

Using the spread operator (`...`) we add all the default config values to our config object.
Then we overwrite these values with the config passed by the programmer but only those that are *present*.

Using these conventions
