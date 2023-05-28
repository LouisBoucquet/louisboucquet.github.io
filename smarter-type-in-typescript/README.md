# Smarter types in typescript

## What are types?

What are types? It kind of depends on who you ask and what language they're using.
To a C or Rust developer a type is propably closely tied to their memory model.
For example a `Person` struct with a `name` and an `age` has to have x amount of bytes allocated to the `name` and y amount of bytes allocated to the `age`.
Then, when getting a pointer and being told that that bit of memory "behaves" as that struct, a C programmer (or rather the compiler) can use that bit of memory as a `Person`.

In this example `Person` is an abstraction to the used memory model to store and process data.

Just as Rust structs are an abstraction to the binary state of the program, typescript types are an abstraction to the state in the javascript runtime.
Both type systems offer an abstraction to handle data at runtime in a more understandable way.
Just like we could write assembly and manipulate memory and state directly, we could write javascript directly.
So why don't we?

When writing assembly or javascript there's no compiler to tell us that "property `x` doesn't exist on type `Y`".
So in escense, we use types so that the compiler can understand the code we write, help us understand the code and tell us when code we are writing is wrong in some way.

Keeping in mind the goal of a type system we might create our types in such a way that gives the compiler a deeper understanding of our code base.
The more the compiler knows about our code, the more it can help us prevent bugs.

The rest of this talk will provide ways of letting the compiler do as much work for you as possible by providing the nescesarry type information.
Examples are based on real production code bases.

## Selected item

Consider the following code:

```ts
type SelectedItem = {
	item?: Item;
	subItem?: SubItem;
};
```

This type represents a selected `item` with possibly a selected `subItem`.
Both `item` and `subItem` are nullable and have 2 possible states which makes the total number of possible states 4:

```ts
const selectedItem1 = {} satisfies SelectedItem;

const selectedItem2 = {
	item: Item,
} satisfies SelectedItem;

const selectedItem3 = {
	item: Item,
	subItem: SubItem,
} satisfies SelectedItem;

const selectedItem4 = {
	subItem: SubItem,
} satisfies SelectedItem;
```

As you may have noticed, it's possible to have a `subItem` without a matching `item`.
This is invalid state.
The reason this invalid state is possible in the first place is because we have an inaccurate type.
We can fix this type by explicitly listing out the possible states:

```ts
type SelectedItem =
	| {}
	| {
			item: Item;
			subItem?: SubItem;
	  };
```

Either nothing is selected, or an `item` is selected with possibly a `subItem`.
We have made invalid state unrepresentable:

```ts
const selectedItem = {
	subItem: SubItem,
} satisfies SelectedItem;
```

Now typescript warns us that this state does not match `SelectedItem`.

## On paper

Old type:

```ts
type Document = {
	pdfUrl: string;
	// ...
};
```

A new feature requires that `pdfUrl` can be optional, based on which type of document it is.
In de documentation we read that `pdfUrl` becomes optional and there's a new field added `type` which can be either `pdf` of `on-paper`.
Like good code monkeys we are, we update our existing type:

```ts
type Document = {
	type: 'pdf' | 'on-paper';
	pdfUrl?: string;
	// ...
};
```

But now consider the following code:

```ts
declare const document: Document;

if (document.type === 'pdf') showPdf(document.pdfUrl);
```

This won't compile (assuming `showPdf` takes a `string`) because `pdfUrl` has become optional.
To make this work you'd either have to check that `pdfUrl` is actually present or just unsafly cast it as a `string`.
The reason we can cast this is because we know that whithin this `if`, the document is of type `pdf` which always has a `pdfUrl`.
So why don't we explain this to typescript, we can model this extra knowledge in our type:

```ts
type Document =
	| {
			type: 'pdf';
			pdfUrl: string;
			// ...
	  }
	| {
			type: 'on-paper';
			// ...
	  };
```

Now this code does compile.

```ts
declare const document: Document;

if (document.type === 'pdf') showPdf(document.pdfUrl);
```

## Enums

### Options array

Consider the following code:

```ts
type SomeEnum = 'a' | 'b';

const someEnumOptions = [
	{ value: 'a', label: 'A' },
	{ value: 'b', label: 'B' },
] satisfies { value: SomeEnum; label: string };
```

There's an enum `SomeEnum` defined and we want to define a list of "options" to pass to an input field.
The problem here is that there's nothing stopping us from adding an extra "option" and having one enum value twice.
Also, when adding an enum value there is nothing to remind you also to add an option to the array.
In this case it might seem obvious but you can imagine that in en giant code base this becomes less and less obvious.
There for we want typescript to remind/help us as much as possible, and a way to do this is as follows:

```ts
const someEnumValues = ['a', 'b'] as const;
type SomeEnum = typeof someEnumValues[number];

const someEnumLabels = {
	a: 'A',
	b: 'B',
} satisfies Record<SomeEnum, string>;


const someEnumOptions = someEnumValues.map((value) => ({
	value,
	label: someEnumLabels[value],
}));
```

Note that I added a `someEnumValues` array, this way of defining an enum gives me access to an array of all enum values.
Secondly instead of defining the labels in an array, which is a data structure for adding as little or as many things you want, I used a record here.
A record is a data structure to map certain keys (enum values) to a corresponding value (labels).
By picking the right data structure typecript now tells us if an element in the record has to be added or has been define twice.
Then a simple `.map(...)` converts to the options we need.

### Exhaustive enum branching

Now that we have an enum type we need a way to check every variation of a type.

```ts
type SomeEnum = 'a' | 'b';

function someFunction(someEnum: SomeEnum): number {
	if (someEnum === 'a') return 1;
	if (someEnum === 'b') return 2;
	return assertNever(someEnum);
}

function assertNever(value: never): never {
	throw new Error('Value was not `never`');
}
```

You can define a util function `assertNever` to check that you already checked every branch of a type's variations.
The function `assertNever` takes an argument with the `never` type.
This way typescript can ensure that all variations have been handled the the remaining type of `someEnum` is `never`.
The return also needs to be `never` to let typescript know that it should not be possible to have none of the if statements match.

## Versioned localstorage demo

steps:

_Chapter 1_

-   Intro, explaining that there will be live coding around local storage
-   Showing the base project (none functional)
-   Explaining that in order to make it work we need to persist the data

_Chapter 2_

-   Implementing with plain untyped localstorage
-   Refactoring value (from `string` to `{ title: string, body: string }`)
-   Explaining that
    -   Typescript doesn't warn us
    -   There's a runtime bug introduced
-   Naive/easy solution: clear local storage

_Chapter 3_

-   Introducing the new package, explaining that:
    -   Typescript will warn us
    -   There won't be a runtime bug, it will be caught at compile time
-   Resetting to base project
-   Implementing persistance using the package
-   Executing the same refactoring
-   Showing the runtime bugs
-   Adding a version to the localstorage interface
-   Happy compiling/working code

_Final thoughts_

As much as possible, you want the "correctness" of your code to be the same at compile time and runtime.
If your code compiles I want to be as certain as possible that it won't contain runtime errors.
Actual behaviour will still need to be tested though.
