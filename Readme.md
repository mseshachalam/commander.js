# Commander.js

[![Build Status](https://api.travis-ci.org/tj/commander.js.svg?branch=master)](http://travis-ci.org/tj/commander.js)
[![NPM Version](http://img.shields.io/npm/v/commander.svg?style=flat)](https://www.npmjs.org/package/commander)
[![NPM Downloads](https://img.shields.io/npm/dm/commander.svg?style=flat)](https://npmcharts.com/compare/commander?minimal=true)
[![Install Size](https://packagephobia.now.sh/badge?p=commander)](https://packagephobia.now.sh/result?p=commander)

  The complete solution for [node.js](http://nodejs.org) command-line interfaces, inspired by Ruby's [commander](https://github.com/commander-rb/commander).  
  [API documentation](http://tj.github.com/commander.js/)

## Installation

    $ npm install commander

## Declaring _program_ variable

Commander exports a global object which is convenient for quick programs.
This is used in the examples in this README for brevity.

```js
const program = require('commander');
program.version('0.0.1');
```

For larger programs which may use commander in multiple ways, including unit testing, it is better to create a local Command object to use.

 ```js
 const commander = require('commander');
 const program = new commander.Command();
 program.version('0.0.1');
 ```


## Options

Options are defined with the `.option()` method, also serving as documentation for the options. Each option can have a short flag (single character) and a long name, separated by a comma or space.

The options can be accessed as properties on the Command object. Multi-word options such as "--template-engine" are camel-cased, becoming `program.templateEngine` etc. Multiple short flags may be combined as a single arg, for example `-abc` is equivalent to `-a -b -c`.

### Common option types, boolean and value

The two most used option types are a boolean flag, and an option which takes a value (declared using angle brackets). Both are `undefined` unless specified on command line.

```js
const program = require('commander');

program
  .option('-d, --debug', 'output extra debugging')
  .option('-s, --small', 'small pizza size')
  .option('-p, --pizza-type <type>', 'flavour of pizza');

program.parse(process.argv);

if (program.debug) console.log(program.opts());
console.log('pizza details:');
if (program.small) console.log('- small pizza size');
if (program.pizzaType) console.log(`- ${program.pizzaType}`);
```

```bash
$ pizza-options -d
{ debug: true, small: undefined, pizzaType: undefined }
pizza details:
$ pizza-options -p
error: option `-p, --pizza-type <type>' argument missing
$ pizza-options -ds -p vegetarian
{ debug: true, small: true, pizzaType: 'vegetarian' }
pizza details:
- small pizza size
- vegetarian
$ pizza-options --pizza-type=cheese
pizza details:
- cheese
```

`program.parse(arguments)` processes the arguments, leaving any args not consumed by the options as the `program.args` array.

### Default option value

You can specify a default value for an option which takes a value.

```js
const program = require('commander');

program
  .option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');

program.parse(process.argv);

console.log(`cheese: ${program.cheese}`);
```

```bash
$ pizza-options
cheese: blue
$ pizza-options --cheese stilton
cheese: stilton
```

### Other option types, negatable boolean and flag|value

You can specify a boolean option long name with a leading `no-` to make it true by default and able to be negated.


```js
const program = require('commander');

program
  .option('-n, --no-sauce', 'Remove sauce')
  .parse(process.argv);

if (program.sauce) console.log('you ordered a pizza with sauce');
else console.log('you ordered a pizza without sauce');
```

```bash
$ pizza-options
you ordered a pizza with sauce
$ pizza-options --sauce
error: unknown option `--sauce'
$ pizza-options --no-sauce
you ordered a pizza without sauce
```
You can specify an option which functions as a flag but may also take a value (declared using square brackets).


```js
const program = require('commander');

program
  .option('-c, --cheese [type]', 'Add cheese with optional type');

program.parse(process.argv);

if (program.cheese === undefined) console.log('no cheese');
else if (program.cheese === true) console.log('add cheese');
else console.log(`add cheese type ${program.cheese}`);
```

```bash
$ pizza-options
no cheese
$ pizza-options --cheese
add cheese
$ pizza-options --cheese mozzarella
add cheese type mozzarella
```

### Custom option processing

You may specify a function to do custom processing of option values. The callback function receives two parameters, the user specified value and the 
previous value for the option. It returns the new value for the option.

This allows you to coerce the option value to the desired type, or accumulate values, or do entirely custom processing.

You can optionally specify the default/starting value for the option after the function.

```js
const program = require('commander');

function myParseInt(value, dummyPrevious) {
  // parseInt takes a string and an optional radix
  return parseInt(value);
}

function increaseVerbosity(dummyValue, previous) {
  return previous + 1;
}

function collect(value, previous) {
  return previous.concat([value]);
}

function commaSeparatedList(value, dummyPrevious) {
  return value.split(',');
}

program
  .option('-f, --float <number>', 'float argument', parseFloat)
  .option('-i, --integer <number>', 'integer argument', myParseInt)
  .option('-v, --verbose', 'verbosity that can be increased', increaseVerbosity, 0)
  .option('-c, --collect <value>', 'repeatable value', collect, [])
  .option('-l, --list <items>', 'comma separated list', commaSeparatedList)
;

program.parse(process.argv);

if (program.float !== undefined) console.log(`float: ${program.float}`);
if (program.integer !== undefined) console.log(`integer: ${program.integer}`);
if (program.verbose > 0) console.log(`verbosity: ${program.verbose}`);
if (program.collect.length > 0) console.log(program.collect);
if (program.list !== undefined) console.log(program.list);
```

```bash
$ custom -f 1e2
float: 100
$ custom --integer 2
integer: 2
$ custom -v -v -v
verbose: 3
$ custom -c a -c b -c c
[ 'a', 'b', 'c' ]
$ custom --list x,y,z
[ 'x', 'y', 'z' ]
```

### Version option

The optional `version` method adds handling for displaying the command version. The default option flags are `-V` and `--version`, and when present the  command prints the version number and exits.

```js
    program.version('0.0.1');
```

```bash
    $ ./examples/pizza -V
    0.0.1
```

You may specify custom flags by passing an additional parameter to the `version` method using the same syntax as the `option` method. The version flags can be named anything, but a long name is required.

```js
program.version('0.0.1', '-v, --version');
```



## Command-specific options

You can attach options to a command.

```js
#!/usr/bin/env node

var program = require('commander');

program
  .command('rm <dir>')
  .option('-r, --recursive', 'Remove recursively')
  .action(function (dir, cmd) {
    console.log('remove ' + dir + (cmd.recursive ? ' recursively' : ''))
  })

program.parse(process.argv)
```

A command's options are validated when the command is used. Any unknown options will be reported as an error. However, if an action-based command does not define an action, then the options are not validated.

## Regular Expression
```js
program
  .version('0.1.0')
  .option('-s --size <size>', 'Pizza size', /^(large|medium|small)$/i, 'medium')
  .option('-d --drink [drink]', 'Drink', /^(coke|pepsi|izze)$/i)
  .parse(process.argv);

console.log(' size: %j', program.size);
console.log(' drink: %j', program.drink);
```

## Variadic arguments

 The last argument of a command can be variadic, and only the last argument.  To make an argument variadic you have to
 append `...` to the argument name.  Here is an example:

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('commander');

program
  .version('0.1.0')
  .command('rmdir <dir> [otherDirs...]')
  .action(function (dir, otherDirs) {
    console.log('rmdir %s', dir);
    if (otherDirs) {
      otherDirs.forEach(function (oDir) {
        console.log('rmdir %s', oDir);
      });
    }
  });

program.parse(process.argv);
```

 An `Array` is used for the value of a variadic argument.  This applies to `program.args` as well as the argument passed
 to your action as demonstrated above.

## Specify the argument syntax

```js
#!/usr/bin/env node

var program = require('commander');

program
  .version('0.1.0')
  .arguments('<cmd> [env]')
  .action(function (cmd, env) {
     cmdValue = cmd;
     envValue = env;
  });

program.parse(process.argv);

if (typeof cmdValue === 'undefined') {
   console.error('no command given!');
   process.exit(1);
}
console.log('command:', cmdValue);
console.log('environment:', envValue || "no environment given");
```
Angled brackets (e.g. `<cmd>`) indicate required input. Square brackets (e.g. `[env]`) indicate optional input.

## Git-style sub-commands

```js
// file: ./examples/pm
var program = require('commander');

program
  .version('0.1.0')
  .command('install [name]', 'install one or more packages')
  .command('search [query]', 'search with optional query')
  .command('list', 'list packages installed', {isDefault: true})
  .parse(process.argv);
```

When `.command()` is invoked with a description argument, no `.action(callback)` should be called to handle sub-commands, otherwise there will be an error. This tells commander that you're going to use separate executables for sub-commands, much like `git(1)` and other popular tools.  
The commander will try to search the executables in the directory of the entry script (like `./examples/pm`) with the name `program-command`, like `pm-install`, `pm-search`.

Options can be passed with the call to `.command()`. Specifying `true` for `opts.noHelp` will remove the subcommand from the generated help output. Specifying `true` for `opts.isDefault` will run the subcommand if no other subcommand is specified.

If the program is designed to be installed globally, make sure the executables have proper modes, like `755`.

### `--harmony`

You can enable `--harmony` option in two ways:
* Use `#! /usr/bin/env node --harmony` in the sub-commands scripts. Note some os version don’t support this pattern.
* Use the `--harmony` option when call the command, like `node --harmony examples/pm publish`. The `--harmony` option will be preserved when spawning sub-command process.

## Automated --help

 The help information is auto-generated based on the information commander already knows about your program, so the following `--help` info is for free:

```  
$ ./examples/pizza --help
Usage: pizza [options]

An application for pizzas ordering

Options:
  -h, --help           output usage information
  -V, --version        output the version number
  -p, --peppers        Add peppers
  -P, --pineapple      Add pineapple
  -b, --bbq            Add bbq sauce
  -c, --cheese <type>  Add the specified type of cheese [marble]
  -C, --no-cheese      You do not want any cheese
```

## Custom help

 You can display arbitrary `-h, --help` information
 by listening for "--help". Commander will automatically
 exit once you are done so that the remainder of your program
 does not execute causing undesired behaviors, for example
 in the following executable "stuff" will not output when
 `--help` is used.

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('commander');

program
  .version('0.1.0')
  .option('-f, --foo', 'enable some foo')
  .option('-b, --bar', 'enable some bar')
  .option('-B, --baz', 'enable some baz');

// must be before .parse() since
// node's emit() is immediate

program.on('--help', function(){
  console.log('')
  console.log('Examples:');
  console.log('  $ custom-help --help');
  console.log('  $ custom-help -h');
});

program.parse(process.argv);

console.log('stuff');
```

Yields the following help output when `node script-name.js -h` or `node script-name.js --help` are run:

```
Usage: custom-help [options]

Options:
  -h, --help     output usage information
  -V, --version  output the version number
  -f, --foo      enable some foo
  -b, --bar      enable some bar
  -B, --baz      enable some baz

Examples:
  $ custom-help --help
  $ custom-help -h
```

## .outputHelp(cb)

Output help information without exiting.
Optional callback cb allows post-processing of help text before it is displayed.

If you want to display help by default (e.g. if no command was provided), you can use something like:

```js
var program = require('commander');
var colors = require('colors');

program
  .version('0.1.0')
  .command('getstream [url]', 'get stream URL')
  .parse(process.argv);

if (!process.argv.slice(2).length) {
  program.outputHelp(make_red);
}

function make_red(txt) {
  return colors.red(txt); //display the help text in red on the console
}
```

## .help(cb)

  Output help information and exit immediately.
  Optional callback cb allows post-processing of help text before it is displayed.


## Custom event listeners
 You can execute custom actions by listening to command and option events.

```js
program.on('option:verbose', function () {
  process.env.VERBOSE = this.verbose;
});

// error on unknown commands
program.on('command:*', function () {
  console.error('Invalid command: %s\nSee --help for a list of available commands.', program.args.join(' '));
  process.exit(1);
});
```

## Examples

```js
var program = require('commander');

program
  .version('0.1.0')
  .option('-C, --chdir <path>', 'change the working directory')
  .option('-c, --config <path>', 'set config path. defaults to ./deploy.conf')
  .option('-T, --no-tests', 'ignore test hook');

program
  .command('setup [env]')
  .description('run setup commands for all envs')
  .option("-s, --setup_mode [mode]", "Which setup mode to use")
  .action(function(env, options){
    var mode = options.setup_mode || "normal";
    env = env || 'all';
    console.log('setup for %s env(s) with %s mode', env, mode);
  });

program
  .command('exec <cmd>')
  .alias('ex')
  .description('execute the given remote cmd')
  .option("-e, --exec_mode <mode>", "Which exec mode to use")
  .action(function(cmd, options){
    console.log('exec "%s" using %s mode', cmd, options.exec_mode);
  }).on('--help', function() {
    console.log('');
    console.log('Examples:');
    console.log('');
    console.log('  $ deploy exec sequential');
    console.log('  $ deploy exec async');
  });

program
  .command('*')
  .action(function(env){
    console.log('deploying "%s"', env);
  });

program.parse(process.argv);
```

More Demos can be found in the [examples](https://github.com/tj/commander.js/tree/master/examples) directory.

## License

[MIT](https://github.com/tj/commander.js/blob/master/LICENSE)
