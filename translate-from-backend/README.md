# Dynamically translating json responses from an api

I've encountered an api where all translations are given in the response json, so naturally I made a way to easily and dynamically get the right tag.
It was important to me that the code was:

* Easy to use
* Dynamic (Selected tag updates if the chosen language updates)

This article requires basic knowledge of [rxjs](https://rxjs.dev).

## Types

Let's start by defining types so we can see what we're working with.

Incomming data:

```ts
interface WithTranslations {
	labelEn: string;
	labelEs: string;
	// All the languages you need ...
}
```

This type describes an object from the api, and it contains all the supported languages.

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
	labelEn: string;
	labelEs: string;
	additionalField: string;
}

// This

type TranslatedEntity = Translate<Entity>;

// Is equivalent to

interface TranslatedEntity {
	label: string;
	additionalField: string;
}
```

With the types defined we need to define a function to transform a `WithTranslations` type to a `Translated<T>` type.

## The translate function

### Setup

Befor we can actually implement the `translate(...)` function, there's some setup needed.

We can use a class to group other variables and functions we'll need.
We'll need a variable `langMapper`, this maps a selected language to the corresponding label.
We'll also need a `selectedLang$` observable.

```ts
type SupportedLang = 'en' | 'es' | /* ... other supported languages */;

class ApiTranslater {
	langMapper: Record<SupportedLang, keyof WithTranslations> = {
		en: 'labelEn',
		es: 'labelEs',
		/* ... other supported languages */
	}

	selectedLang$: Observable<SupportedLang> = /* ... */;
}
```

The type `SupportedLang` defines all the languages you want to support.

### Implementation of `translateWithLang(...)`

With the setup out of the way we can finally begin with the implementation.

```ts
class ApiTranslater {
	/* ... */

	translateWithLang<T extends WithTranslation>(
		toTranslate: T,
		selectedLang: SupportedLang,
	): Translated<T> {
		const labelKey = this.langMapper[selectedLang];

		return {
			...toTranslate,
			label: toTranslate[labelKey],
		}
	}
}
```

What actually happens in this bit of code?
The function accepts an object to translate and the language to translate to.
First we get the key of the actual label we need, the mapping between a language and the corresponding label has been defined in `this.langMapper`.
Next we copy every field of the original object and add a field called `label`, this contains the content of the correct label.

Note that we don't actually remove the old labels, they are still present in the returned object.
There's no need to remove these labels because the type hides the labels, an IDE won't show them as options.

### Implementation of `translate(...)`

To use `translateWithLang(...)` you need to know the selected language.
We can actually abstract this away to make things easier.

```ts
class ApiTranslater {
	/* ... */

	translate<T extends WithTranslation>(
		toTranslate: T,
	): Observable<Translated<T>> {
		return this.selectedLang$
			.pipe(
				map(selectedLang => this.translateWithLang(toTranslate, selectedLang)),
			);
	}
}
```

Using rxjs' pipe system we can map from the `selectedLang$`.
This means we don't need to know the selected language when calling the `translate(...)` function.

### Extra utility functions

To facillitate every need we can define some extra utility functions.

```ts
class ApiTranslater {
	/* ... */

	translateArray<T extends WithTranslation>(
		toTranslateArray: T[],
	): Observable<Translated<T>[]> {
		return this.selectedLang$.pipe(
			map(lang =>
				toTranslateArray.map(toTranslate =>
					this.translateWithLang(toTranslate, lang))),
		);
	}

	translatePipe<T extends WithTranslation>():
		(source$: Observable<T>) =>
			Observable<Translated<T>> {
		return source$ => source$.pipe(switchMap(toTranslate => this.translate(toTranslate)));
	}

	translateArrayPipe<T extends WithTranslation>():
		(source$: Observable<T[]>) =>
			Observable<Translated<T>[]> {
		return source$ => source$.pipe(switchMap(toTranslate => this.translateArray(toTranslate)));
	}
}
```

`translateArray` translates an entire array. `translatePipe`, `translateArrayPipe` are "pipe" variations of existing functions.

## `translate` pipe
