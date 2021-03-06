# lodash-modularize [![Dependency Status](https://david-dm.org/megawac/lodash-modularize.svg)](https://david-dm.org/megawac/lodash-modularize)

Lodash is starting to get pretty heafty; this is a tool to generate modular lodash builds so lodash only includes what you use. This can lead to faster startup and smaller builds (when using `compile`, `browserify`, `r.js`, etc).

If you're using `babel` checkout [`babel-plugin-lodash`](https://github.com/megawac/babel-plugin-lodash) for a different approach.

## Whats in the Box

<a href="http://www.youtube.com/watch?feature=player_embedded&v=o9GrUNAqwNY
" target="_blank"><img src="http://img.youtube.com/vi/o9GrUNAqwNY/0.jpg"
alt="IMAGE ALT TEXT HERE" width="300" height="180" border="10" /></a>

### Features

- Detect lodash methods/modules being used in source code
- Compile a perfect [custom lodash build using the cli](https://lodash.com/custom-builds)
- Compile a custom modular `lodash.js` which imports the exact modules you use
- Update references (e.g.) `require('lodash')` to `require('./src/custom-lodash'`
- Supports AMD, CJS, ES6 and UMD
- Natural recompilation: if you decide to output a build (updating references), the tool recognizes `lodash` features coming from the output path (e.g. `lib/lodash.js`) in addition to regular lodash sources.
- Supports using [lodash npm modules](https://www.npmjs.com/browse/keyword/lodash-modularized)
- Other sweetness (see below and try `lodash-modularize --help`)

### Example Usage

All examples are taken from this project

```sh
# List all the method's being used in src
lodash-modularize src/** --list
# => assign,chain,flatten,includes,isArray,reject,result,template,uniq,zipObject

lodash-modularize src/**
# <formated module>

lodash-modularize src/** -o src/depends/lodash.js

lodash-modularize src/** -o src/depends/lodash.js --format es6

# Set the global variable to search for to `lodash`
lodash-modularize src/** -o src/depends/lodash.js --global lodash --exports umd

# Compile the code using lodash-cli!
lodash-modularize src/** -o src/depends/lodash.js --amd --compile

# Update the projects using lodash to use the built depends/lodash instead
lodash-modularize src/** -o depends/lodash.js --update
```

##### Fancy `package.json` script

Use this tool to easily use [lodash npm modules](https://www.npmjs.com/browse/keyword/lodash-modularized) and avoid installing the entire lodash package (set `lodash` as a dev dep)!

```js
{
    "devDependencies": {"lodash": "^3.0", "lodash-modularize": "^1.0"},
    "scripts": {
        "prepublish": "lodash-modularize src/**.js -o src/depends/lodash.js -u --use-npm-modules --install-npm-modles"
    }
}
```

### So what can it detect

**app.js**
```js
import _, {sortBy, uniq} from 'lodash';
let log = require('logger'),
    lodash = require('lodash');

let result = sortBy(_.flatten(uniq([{a: 1}, {a: 2}, {a: 1}, {a: 0}])), 'a');
lodash.each(result, log);
```

```sh
$ lodash-modularize app.js --list
# => each, flatten, sortBy, uniq

$ lodash-modularize ./test/sample.js --cjs -o lodash.js
```
**lodash.js**
```js
var each = require('lodash/collection/each');
var flatten = require('lodash/array/flatten');
var sortBy = require('lodash/collection/sortBy');
var uniq = require('lodash/array/uniq');

function lodash() {
  throw 'lodash chaining is not included in this build... Try rebuilding.';
}
module.exports = lodash;

lodash.each = each;
lodash.flatten = flatten;
lodash.sortBy = sortBy;
lodash.uniq = uniq;
```

And many other patterns including globals (opt-in), **chaining**, and mixins.

## Notes

Lazy chaining is not fully supported (it works but its not lazy).

You should use in conjunction with linters (jshint/eslint/etc) as this won't detect unused variables.

All though we go out of our way to be robust and support various ways to detect lodash imports of lodash there are things we don't bother to handle. For example if you do any of these things, we'll probably miss it (same goes for global variables)

```js
const _ = require('lodash');
const lodash = _;

function reassignment() {
    let _ = 'reassigned'.trim();
    return _;
}
```
