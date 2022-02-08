# Dynamically translating json responses from an api

I've encountered an api where all translations are given in the response json, so naturally I made a way to easily and dynamically get the right tag.
It was important to me that the code was:

* Easy to use
* Dynamic (Selected tag updates if the chosen language updates)

## Types

Let's start by defining types so we can see what we're working with.

Incomming data:

```ts
interface WithTranslations {
	labelEn: string,
	labelEs: string,
	// All the languages you need ...
}
```

This type describes an object from the backend, and it contains all the supported languages.

Transformed data:

```ts
type Translated<T extends WithTranslations> =
	Omit<T, keyof WithTranslations> & {
		label: string
	}
```

There's a few things to unpack here:

The type `Omit<T, Keys>` removes all the `Keys` from `T`. In this case we use it to remove all language-annotated labels, we won't need it anymore.

Once we removed all the labels we don't need, it's time to add one label which will contain the content of the label who's language is selected.

Note that we omit the `keyof WithTranslations` keys and not `keyof T`. Because of this we can still access any other fields that might have been present. For example:

```ts
interface Entity extends WithTranslations {
	labelEn: string,
	labelEs: string,
	additionalField: string,
}

// This

type TranslatedEntity = Translate<Entity>;

// Is equivalent to

interface EnTranslatedEntitytity {
	label: string,
	additionalField: string,
}
```

With the types defined we need to define a function to transform a `WithTranslations` type to a `Translated<T>` type.
