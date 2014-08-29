# SimpleSearch

SimpleSearch is becoming a powerful JavaScript tool to filter an array for matched (and approximately matched) strings.

## Installation and Setup

We use UMD to maintain compatibility with several popular module systems.

### CommonJS Setup (for environments like Node.js and Browserify)

Install SimpleSearch...

```
npm install --save simplesearch
```

...and require it in your project

```javascript
var SimpleSearch = require('simplesearch');
SimpleSearch.filter(...);
```

### AMD Setup (for Require.JS)

**Disclaimer:** it's been awhile since I've used Require.JS, and I'm no expert. This method is currently untested, but it should work.

Install SimpleSearch by downloading a [tagged release](https://github.com/mlms13/SimpleSearch/releases). Or you could install with npm and dig it out of the `node_modules` folder... I don't really know the preferred way to do package management in AMD environments.

Then, start using it like this:

```javascript
require(['path/to/simplesearch.js'], function(SimpleSearch) {
  SimpleSearch.filter(...);
});
```

### No module loader? No problem.

If you're just looking to dump a script into your page, we can work with that. Start by [downloading the latest release](https://github.com/mlms13/SimpleSearch/releases).

```html
<script src="/path/to/simplesearch.js"></script>
<script>
  window.SimpleSearch.filter(...);
</script>
```

## Usage

The SimpleSearch object defines two useful methods related to filtering and matching text.

### SimpleSearch.matches(item, search)

Given an item to test against and a search string to look for, return true
or false, depending on whether the match is found, according to our criteria:
 - The search string can leave out characters
 - Search string cannot get character order wrong
 - Ignore spaces in the search string
 - Ignore other non-alphanumeric characters in the search string
 - Word order of the target item doesn't matter

**Parameters**

- **item**: `string`, The string to be searched for matches

- **search**: `string`, The string of characters we are searching for

**Returns**: `Boolean`

**Example**:
```js
SimpleSearch.matches('foobar', 'fbr'); // returns true
SimpleSearch.matches('foobar', 'bf'); // returns false
SimpleSearch.matches('foo', 'f oo!'); // returns true
SimpleSearch.matches('bar (foo)', foobar); // returns true
```


### SimpleSearch.filter(data, search)

Given an array as the first paramater and a string of search text as the
second, filter() will return a new array that is a subset of the original.
The results are filtered according to the criteria listed for matches().

**Parameters**

- **data**: `Array.&lt;string&gt;`, Array of strings that will be filtered

- **search**: `string`, The string of characters we are searching for

**Returns**: `Array.&lt;string&gt;`, - A new array, a subset of the original data

**Example**:
```js
SimpleSearch.filter(["foo", "bar", "baz"], "ba"); // returns ["bar", "baz"]
```


## Roadmap

The `matches` function has some hard-coded rules about how spaces, special characters, text case, and word order work.  All of that will soon be configurable.

Filter currently only handles arrays of strings. Filtering an array of objects is coming next.

```javascript
// data as an array of objects instead of strings
var data = [{
  name: "berry",
  alias: ["strawberry", "acai", "blueberry"]
}, {
  name: "chickpea",
  alias: "garbanzo bean"
}, {
  name: "apple, raw"
}];
```

Instead of passing an array and a search string to `filter`, you will be able to pass a big configuration object.

```javascript
// filter(obj)
SimpleSearch.filter({
  data: data,
  fields: ["name", "alias"], // fields to be searched, ordered by preference.
                             // we only search "alias" if "name" doesn't match
  exactMatch: false, // if set to true, "ba" would not match "bean"
  splitWords: true, // if set to false, "bean garb" wouldn't match "garbanzo bean"
  punctuation: false, // if set to true, "(raw)" wouldn't match "raw"
  success: function (match) {
    // a callback function to be run for each matching item
    // receives a `match` object with the following properties
    // - `field` -- the field in which the match was found
    // - `original` -- original object (from data array) in which match was found
    // - `charPos` -- array of indexes of matched characters
    //                e.g. [0, 1, 3] for "bean" given search string "ben"
  },
  fail: function (original) {
    // callback to be run if match fails
    // receives the original object that was tested for a match
  }
});
```

Farther future, we're looking into the Levenshtein algorithm to do fuzzy matching. This way, we'll be able to:

- Give each match a score, based on similarity to the search term
- Sort by score before returning the filtered array (and before calling success/fail callback functions)
