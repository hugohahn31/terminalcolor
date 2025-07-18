## Description
The fastest Node.js library for formatting terminal text with ANSI colors~!

## Features

* No dependencies
* Super [lightweight](#load-time) & [performant](#performance)
* Supports [nested](#nested-methods) & [chained](#chained-methods) colors
* No `String.prototype` modifications
* Conditional [color support](#conditional-support)
* [Fully treeshakable](#individual-colors)
* Familiar [API](#api)

---

As of `v3.0` the Chalk-style syntax (magical getter) is no longer used.<br>Please visit [History](#history) for migration paths supporting that syntax.

---


## Install

```
$ npm install terminalcolor
```


## Usage

```js
import terminalcolor from 'terminalcolor';

// basic usage
terminalcolor.red('red text');

// chained methods
terminalcolor.blue().bold().underline('howdy partner');

// nested methods
terminalcolor.bold(`${ white().bgRed('[ERROR]') } ${ terminalcolor.red().italic('Something happened')}`);
```

### Chained Methods

```js
const { bold, green } = require('terminalcolor');

console.log(bold().red('this is a bold red message'));
console.log(bold().italic('this is a bold italicized message'));
console.log(bold().yellow().bgRed().italic('this is a bold yellow italicized message'));
console.log(green().bold().underline('this is a bold green underlined message'));
```

### Nested Methods

```js
const { yellow, red, cyan } = require('terminalcolor');

console.log(yellow(`foo ${red().bold('red')} bar ${cyan('cyan')} baz`));
console.log(yellow('foo ' + red().bold('red') + ' bar ' + cyan('cyan') + ' baz'));
```


### Conditional Support

Toggle color support as needed; `terminalcolor` includes simple auto-detection which may not cover all cases.

> **Note:** Both `terminalcolor` and `terminalcolor/colors` share the same detection logic.

```js
import terminalcolor from 'terminalcolor';

// manually disable
terminalcolor.enabled = false;

// or use another library to detect support
terminalcolor.enabled = require('color-support').level > 0;

console.log(terminalcolor.red('I will only be colored red if the terminal supports colors'));
```

> **Important:** <br>Colors will be disabled automatically in non [TTY contexts](https://nodejs.org/api/process.html#process_a_note_on_process_i_o). For example, spawning another process or piping output into another process will disable colorization automatically. To force colors in your piped output, you may do so with the `FORCE_COLOR=1` environment variable:

```sh
$ node app.js #=> COLORS
$ node app.js > log.txt  #=> NO COLORS
$ FORCE_COLOR=1 node app.js > log.txt #=> COLORS
$ FORCE_COLOR=0 node app.js > log.txt #=> NO COLORS
```

## API

Any `terminalcolor` method returns a `String` when invoked with input; otherwise chaining is expected.

> It's up to the developer to pass the output to destinations like `console.log`, `process.stdout.write`, etc.

The methods below are grouped by type for legibility purposes only. They each can be [chained](#chained-methods) or [nested](#nested-methods) with one another.

***Colors:***
> black &mdash; red &mdash; green &mdash; yellow &mdash; blue &mdash; magenta &mdash; cyan &mdash; white &mdash; gray &mdash; grey

***Backgrounds:***
> bgBlack &mdash; bgRed &mdash; bgGreen &mdash; bgYellow &mdash; bgBlue &mdash; bgMagenta &mdash; bgCyan &mdash; bgWhite

***Modifiers:***
> reset &mdash; bold &mdash; dim &mdash; italic* &mdash; underline &mdash; inverse &mdash; hidden &mdash; strikethrough*

<sup>* <em>Not widely supported</em></sup>


## Individual Colors

When you only need a few colors, it doesn't make sense to import _all_ of `terminalcolor` because, as small as it is, `terminalcolor` is not treeshakeable, and so most of its code will be doing nothing. In order to fix this, you can import from the `terminalcolor/colors` submodule which _fully_ supports tree-shaking.

The caveat with this approach is that color functions **are not** chainable~!<br>Each function receives and colorizes its input. You may combine colors, backgrounds, and modifiers by nesting function calls within other functions.

```js
// or: import * as terminalcolor from 'terminalcolor/colors';
import { red, underline, bgWhite } from 'terminalcolor/colors';

red('red text');
//~> terminalcolor.red('red text');

underline(red('red underlined text'));
//~> terminalcolor.underline().red('red underlined text');

bgWhite(underline(red('red underlined text w/ white background')));
//~> terminalcolor.bgWhite().underline().red('red underlined text w/ white background');
```

> **Note:** All the same [colors, backgrounds, and modifiers](#api) are available.

***Conditional Support***

The `terminalcolor/colors` submodule also allows you to toggle color support, as needed.<br>
It includes the same initial assumptions as `terminalcolor`, in an attempt to have colors enabled by default.

Unlike `terminalcolor`, this setting exists as `terminalcolor.$.enabled` instead of `terminalcolor.enabled`:

```js
import * as terminalcolor from 'terminalcolor/colors';
// or: import { $, red } from 'terminalcolor/colors';

// manually disabled
terminalcolor.$.enabled = false;

// or use another library to detect support
terminalcolor.$.enabled = require('color-support').level > 0;

console.log(red('I will only be colored red if the terminal supports colors'));
```


## Benchmarks

> Using Node v10.13.0

### Load time

```
chalk        :: 5.303ms
terminalcolor        :: 0.488ms
terminalcolor/colors :: 0.369ms
ansi-colors  :: 1.504ms
```

### Performance

```
# All Colors
  ansi-colors      x 177,625 ops/sec ±1.47% (92 runs sampled)
  chalk            x 611,907 ops/sec ±0.20% (92 runs sampled)
  terminalcolor            x 742,509 ops/sec ±1.47% (93 runs sampled)
  terminalcolor/colors     x 881,742 ops/sec ±0.19% (98 runs sampled)

# Stacked colors
  ansi-colors      x  23,331 ops/sec ±1.81% (94 runs sampled)
  chalk            x 337,178 ops/sec ±0.20% (98 runs sampled)
  terminalcolor            x  78,299 ops/sec ±1.01% (97 runs sampled)
  terminalcolor/colors     x 104,431 ops/sec ±0.22% (97 runs sampled)

# Nested colors
  ansi-colors      x  67,181 ops/sec ±1.15% (92 runs sampled)
  chalk            x 116,361 ops/sec ±0.63% (94 runs sampled)
  terminalcolor            x 139,514 ops/sec ±0.76% (95 runs sampled)
  terminalcolor/colors     x 145,716 ops/sec ±0.97% (97 runs sampled)
```


## History

This project originally forked [`ansi-colors`](https://github.com/doowb/ansi-colors).

Beginning with `terminalcolor@3.0`, the Chalk-style syntax (magical getter) has been replaced with function calls per key:

```js
// Old:
c.red.bold.underline('old');

// New:
c.red().bold().underline('new');
```
> <sup><em>As I work more with Rust, the newer syntax feels so much better & more natural!</em></sup>

If you prefer the old syntax, you may migrate to `ansi-colors` or newer `chalk` releases.<br>Versions below `terminalcolor@3.0` have been officially deprecated.


## License

terminalcolor is licensed under the terms of the MIT License. See the [license](license) file for details.