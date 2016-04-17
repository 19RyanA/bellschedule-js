BellSchedule-js
---------------

This is a simple JavaScript utility to fetch the schedule from [HarkerDev/bellschedule](https://github.com/HarkerDev/bellschedule) and interact with it.

### Downloading

There are two versions of the script.
* [bellschedule.js](bellschedule.js) is the raw version, written using ES2015 syntax and APIs. As ES2015 is relatively new, this won't work in a lot of browsers (see below for estimated compatibility).
* [bellschedule-compiled.js](bellschedule-compiled.js) is precompiled using [Babel](http://babeljs.io/) and includes the [Babel polyfill](http://babeljs.io/docs/usage/polyfill/) for ES2015 classes such as Map and Promise. It's a lot bigger (about 100 kb vs 8 kb), but should support every common browser out there.

#### Estimated compatibility (from http://kangax.github.io/compat-table/es6/) and testing

|     Edge     	|                 Firefox                	|                Chrome               	|                      Safari                      	| Any mobile browser 	|
|:------------:	|:--------------------------------------:	|:-----------------------------------:	|:------------------------------------------------:	|--------------------	|
| 13+ (latest) 	| None - doesn't like `for (const x of y)` 	| 49+ (old version; latest will work) 	| None - doesn't support a load of ES2015 features 	| Most likely not    	|

So, if you want to support any browser that isn't Chrome or Edge, I'd suggest using the compiled version :cry:.

### Usage

The script exports a single object, `bellSchedule`, used to interact with the schedule.
Below are its properties and methods.

#### `index`
A `Map` mapping days of the week with special schedules to their schedules

The keys of the map are timestamps of the day that the special schedule is in effect (i.e. `.getTime()` is called on the date).

Below is a  representation of the values of the map:

```typescript
interface IndexValue {
  mapping: Map<string, string>; // First is the old name of period, second is the new name
  schedule: string; // The name of the schedule to use for this day
}
```

You shouldn't need to use this, as the library provides functions that do so for you.

#### `schedules`
A `Map` mapping the names of schedules to their periods (and the periods' start and end times)

The keys are the names of the schedules, which are strings. The values are of the format below:

```typescript
interface Period { // The interface for periods that make up the schedule
  type: 'period'; // To designate this as a period and not a group
  name: string;
  start: string; // The start time, in form h:mm
  end: string; // Same format as the start time
}
interface Group { // Interface for when Period 5 and 6 intersect
  type: 'group';
  tracks: Period[][]; // An array for each track then an array of the periods in the track
}
type ScheduleValue = (Period|Group)[];
```

#### `refresh()`
Returns a promise that resolves when it successfully fetches the schedule and rejects if a network error occurs.
The resolve function returns an array with `[index: Map, schedule: Map]`, which are just the `index` and `schedule` properties.

#### `for(date)`
Returns the schedule for the given day, where `day` is of type `Date`

The time of the `day` argument does not matter, so to get the schedule for today you can just call the following:
```javascript
bellSchedule.for(new Date())
```

The function returns an object with the following properties:
```typescript
interface forFunctionReturnValue {
  special: boolean; // Whether this is a special schedule or not
  schedule: ModifiedScheduleValue;
}
```

The `ModifiedScheduleValue` type is the type of a value from the `schedules` map, but with the following exceptions:

  * The names of the periods are changed according to the mapping
  * The start and end times are converted into `Date`s

#### `at(date)`
Returns a generator that yields the periods intersecting `date` of type `Date`.
In other words, it will yield periods whose `start <= date < end`.

Ther period will be taken from whatever `for(date)` returns, as shown below:
```typescript
interface Period {
  type: 'period'; // You can ignore this
  name: string;
  start: Date;
  end: Date;
}
```

If you are using the compiled version, you could convert the generated values to an array by doing the following:

```javascript
Array.from(bellSchedule.at(new Date()))
```
In ES2015, you could use the above code or also use the spread operator as shown below:
```javascript
[...bellSchedule.at(new Date())]
```

You can also just loop though the periods by using the following in (almost) any browser...
```javascript
var periods = bellSchedule.at(new Date());
var period = periods.next().value;
while (period) {
  // Do something with the period here
  period = periods.next().value;
}
```
... or by doing the following with ES2015 syntax
```javascript
for (const period of bellSchedule.at(new Date())) {
  // Do whatever you want here
}
```

There's also the function `periods(schedule)` that returns a generator for the periods in a schedule, but it has no practical use besides being used in the `at(date)` function.