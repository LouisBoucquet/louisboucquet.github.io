# Perfect harmony

In western music we are used to using "Equal temperament" which is an awesome system but it has some flaws.
Equal temperament was design to be flexible, you can tune your instrument to equal temperament and it wil sound the same in all 12 keys.
However this ease of use has some drawbacks, every interval you play is a little bit wrong compared to what is mathematically should be.

## Harmonic series

To understand why certain harmonic intervals sound good you need to understand what the harmonic series are.
I won't be covering this here.

## Vocabular

To easily explain what I have to say I'll propose a few conventions I'll use throughout the text.

### Perfect harmony

* Frequency is the amount of times a note vibrates in a second and is expressed in `Hz`
* Harmony is the interaction of multiple notes played together.
* An interval is the distance between 2 notes, expressed as a ratio i.e. 3:2
* An interval is "perfect" when the ratio is a whole fraction
* An interval is "imperfect" when the ration cannot be expressed as a fraction of whole numbers (i.e. PI)

### Equal temperament stuff

* An octave is the perfect interval 2:1
* A half tone is one twelfth of an octave
* A hole tone is one sixth of an octave or 2 half tones
* A cent is a 100th of a half tone
* A (diatonic) key is a collection of 7 of these 12 half tones
* A scale of a key is these 7 notes in ascending order with a root note
* Within a scale we speak of diatonic intervals:
	* A minor and major second are respectively 1 and 2 half tones
	* A minor and major third are respectively 3 and 4 half tones
	* A fourth is 5 half tones
	* A fifth is 7 half tones
	* A major and minor 6 are respectively 8 and 9 half tones
	* A major and minor 7th are respectively 10 and 11 half tones

## How to make perfect harmony

### What is equal temperament?

I defined a half tone to be a twelfth of an octave but glossed over some of the details.
What does it mean to take fractions of intervals?
First we need to look at how we can add intervals on top of each other.

#### Adding intervals

Let's take the `5:4` interval and add it to a note of `100Hz`.
The way this works is you multiply the original note frequency by the ratio.
This would give us `100Hz * 5:4 = 125Hz`.
If we now apply another interval `6:5`, we get `125Hz * 6:5 = 150Hz`.

Adding the two intervals individualy we get `150Hz` which forms a ratio to `100Hz`, `3:2` which is the same as `5:4 * 6:5`.
We see that `(100Hz * 5:4) * 6:5 = 100Hz * (5:4 * 6:5)`

In other words we can "add" interval ratios together (before applying it to a note) by multiplying the ratios.

#### Dividing intervals

Now that we can add intervals we can do the opposite, take an interval and split it in to sub intervals that add back up to the original interval.
When splitting an interval in 2 we are looking for `a * b = i` where `i` is our original interval.
For `i = 3:2`, `a` and `b` could be `5:4` and `6:5`.
This is easy to find and verify but we need a generic way to split an interval in to an arbitrary number of equal intervals.
In the previous example we used exactly 2 intervals which didn't have to be the same.
Now we're looking for `a^n = i` which means `a = i^(1/n)`.

#### 12-tone equal temperament

Equal temperament chooses to divide an octave (2:1) in equal sub intervals.
In western music this division use 12 equal sub intervals or `2^(1/12)`.
Why 12? I don't know some people like to use different divisions. 12-tone does have reasonably small error compared to perfect harmonies.

### Counteracting 12-tone's imperfections

Every note in 12-tone equal temperament is near to a perfect harmony but not exactly the same.
To counteract these mistakes I'll list how far off every half tone is expressed in cents.

To calculate how far a note is off I'll use this formula:

`correction = log(ratio) / log(2^(1/1200)) - cents`

#### The intervals

| Interval | Half tones | Perfect harmony | Correction |
| :- | :-: | :-: | :-: |
| minor 2nd | 1 | 17:16 | +5 cents |
| major 2nd | 2 | 9:8 | +4 cents |
| minor 3rd | 3 | 19:16 | -2 cents |
| major 3rd | 4 | 5:4 | -14 cents |
| major 4th | 5 | 4:3 | -2 cents |
| major 5th | 7 | 3:2 | +2 cents |
| minor 6th | 8 | 25:16 | -27 cents |
| minor 6th | 8 | 13:8 | 41 cents |
| major 6th | 9 | 27:16 | +6 cents |
| minor 7th | 10 | 7:4 | -31 cents |
| major 7th | 11 | 15:8 | -12 cents |
