# Dynamically translating json responses from an api

Most http api's require you to specify the language in which in you want the result to be in.
Some api's, however, give all languages.
This article gives a few ways to deal with responses from such api's in a clean way, keeping following things in mind:

* Easy to use
* Dynamic (Selected tag updates if the chosen language updates)
* Correct typing for good intellisense

This article requires basic knowledge of [rxjs](https://rxjs.dev).

## Types

Let's start by defining some types so we can see what we're working with.

Incoming data:

```ts
interface WithTranslations {
	labelEn: string;
	labelEs: string;
	// All the languages you need ...
}
```

This type describes an object from the api and it contains all the supported languages.

Transformed data:

```ts
type Translated<T extends WithTranslations> =
	Omit<T, keyof WithTranslations> & {
		label: string
	}
```

There's a few things to unpack here:

The type `Omit<T, Keys>` removes all the `Keys` from `T`. In this case we use it to remove all language-annotated labels, we won't need them anymore.

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

With the types defined we need to define a function to transform a `WithTranslations` type into a `Translated<T>` type.

## The translate function

### Setup

Before we can actually implement the `translate(...)` function, there's some setup needed.

We can use a class to group variables and functions we'll need.
We'll need a variable `langMapper`, this maps a language to it's corresponding label.
We'll also need a `selectedLang$` observable, this contains the selcted language.

```ts
type SupportedLang = 'en' | 'es' | /* ... other supported languages */;

class ApiTranslater {
	private langMapper: Record<SupportedLang, keyof WithTranslations> = {
		en: 'labelEn',
		es: 'labelEs',
		/* ... other supported languages */
	}

	private selectedLang$: Observable<SupportedLang> = /* ... */;
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

To facillitate every need, we can define some extra utility functions.

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

`translateArray` translates an entire array. `translatePipe` and `translateArrayPipe` are "pipe" variations of existing functions.

## `translate` pipe

This part is aimed specifically at angular developers, it leverages it's pipes (not to be confused with rxjs pipes). We can make a pipe that translates an object directly within the html.

```ts
@Pipe({
	name: 'translate',
})
export class TranslatePipe implements PipeTransform {
	constructor(
		private readonly translater: ApiTranslater,
		private readonly asyncPipe: AsyncPipe,
	) {}

	// More specific typing
	transform(toTranslate: null | undefined): null;
	transform<T extends BackendTranslated>(
		toTranslate: T,
	): BackendTranslatedLocal<T>;
	transform<T extends BackendTranslated>(
		toTranslate: T | null | undefined,
	): BackendTranslatedLocal<T> | null;

	transform<T extends BackendTranslated>(
		toTranslate: T | null | undefined,
	): BackendTranslatedLocal<T> | null {
		if (toTranslate === null || toTranslate === undefined) return null;

		const translatedObservable$ = this.translater.translate(toTranslate);
		const asyncResult = this.asyncPipe.transform(translatedObservable$);

		return asyncResult;
	}
}
```

We inject the async pipe because you need to append it to an observable anyway, we might as well do that here and save the programmer some typing.

The pipe can be used in an angular template as follows:

```html
{% raw %}<p>{{ (withTranslation | translate).label }}</p>
<p>{{ (withTranslationOptional | translate)?.label }}</p>{% endraw %}
```

## Wrapping up

We've seen how to take an object with labels in multiple languages and pick the right label based on a selected language.
We did this while maintaing proper types and keeping the translation updated with the selected language via an rxjs observable.

A lot of api's give (or even require) it's users to set a "language header".
The api then returns data that has been translated in the backend.
This has some benefits and drawbacks.

* Receiving neatly translated data from an api means that the code handling the data can be a lot simpler, no need to create translating functions when your data is already translated.
* Receiving every translation adds extra overhead to the api, depending on how many languages are supported this might become significant.
* Receiving every translation gives the option to the receiver to switch translations on the fly without having to make a new (costly) call to the api.

If you ever happen to cross an api that sends all translations you'll be better equiped to handle the responses.
