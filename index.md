---
layout: default
title:  PsySH
---
<a class="section-head" id="usage"></a>

## Usage

### PsySH as a REPL

PsySH functions as a <abbr title="Read-Eval-Print Loop">REPL</abbr> for PHP right out of the box! Once you've [installed PsySH](#install), running it directly (`psysh` or `./psysh`) will drop you into an interactive prompt, like so:

```
~ $ ./psysh
Psy Shell v0.1.0-dev (PHP 5.4.9-4ubuntu2.2 — cli) by Justin Hileman
>>>
```

From here, you can type PHP code and see the result interactively:

```
>>> function timesFive($x) {
...     $result = $x * 5;
...     return $result;
... }
=> null
>>> timesFive(10);
=> 50
>>>
```

### PsySH as a Debugger

To use PsySH as a debugger, install it as a Composer dependency or include the Phar directly in your project:

```php
<?php
require('/path/to/psysh');
?>
```

Then, drop this line into your script where you'd like to have a breakpoint:

```php
<?php
eval(\Psy\sh());
?>
```

… which is just a shorter way of saying:

```php
<?php
extract(\Psy\Shell::debug(get_defined_vars()));
?>
```

When your script reaches this point, execution will be suspended and you'll be dropped into a PsySH shell. Your program state is loaded and available for you to inspect and experiment with.

Pro Tip™: You don't have to use `get_defined_vars`… You can pass anything you want in as your debugging context:

```php
<?php
$result = \Psy\Shell::debug(['app' => $myApp]);
?>
```

If you're starting the debug shell from inside a class context, you can pass an optional second argument to add a bound object to the shell. This is super useful because you'll be able to call things on `$this` inside your debug shell, and you will have full access to private and protected members of your current context:

```php
<?php
\Psy\Shell::debug(get_defined_vars(), $this);
?>
```

If you call the shortcut `eval(\Psy\sh())` from inside a class context, you'll get `$this` bound for free.

### Magic Variables

The result of the last (successful) statement is available as `$_`, and the last error or exception is available as `$_e`. You can use these just like you'd use any other variable:

```
>>> 'wat'
=> "wat"
>>> $_
=> "wat"
>>> throw new Exception($_)
Exception with message 'wat'
>>> $_e
=> <Exception #000000004db607ea000000017892abf1> {
       message: "wat",
       file: "phar:///psysh/src/Psy/ExecutionLoop/Loop.php(67) : eval()'d code",
       line: 1
   }
>>>
```

<a class="section-head" id="configure"></a>

## Configuration

While PsySH strives to detect the right settings automatically, you might want to configure it yourself. Just add a file to `~/.config/psysh/config.php`:

```php
<?php

return array(

    // In PHP 5.4+, PsySH will default to your `cli.pager` ini setting. If this
    // is not set, it falls back to `less`. It is recommended that you set up
    // `cli.pager` in your `php.ini` with your preferred output pager.
    //
    // If you are running PHP 5.3, or if you want to use a different pager only
    // for Psy shell sessions, you can override it here.
    'pager' => 'more',

    // Sets the maximum number of entries the history can contain.
    // If set to zero, the history size is unlimited.
    'historySize' => 0,

    // If set to true, the history will not keep duplicate entries.
    // Newest entries override oldest.
    // This is the equivalent of the HISTCONTROL=erasedups setting in bash.
    'eraseDuplicates' => false,

    // By default, PsySH will use a 'forking' execution loop if pcntl is
    // installed. This is by far the best way to use it, but you can override
    // the default by explicitly enabling or disabling this functionality here.
    'usePcntl' => false,

    // PsySH uses readline if you have it installed, because interactive input
    // is pretty awful without it. But you can explicitly disable it if you hate
    // yourself or something.
    'useReadline' => false,

    // PsySH automatically inserts semicolons at the end of input if a statement
    // is missing one. To disable this, set `requireSemicolons` to true.
    'requireSemicolons' => true,

    // While PsySH respects the current `error_reporting` level, and doesn't throw
    // exceptions for all errors, it does log all errors regardless of level. Set
    // `errorLoggingLevel` to 0 to prevent logging non-thrown errors. Set it to any
    // valid `error_reporting` value to log only errors which match that level.
    'errorLoggingLevel' => E_ALL & ~E_NOTICE,

    // "Default includes" will be included once at the beginning of every PsySH
    // session. This is a good place to add autoloaders for your favorite
    // libraries.
    'defaultIncludes' => array(
        __DIR__ . '/include/bootstrap.php',
    ),

    // While PsySH ships with a bunch of great commands, it's possible to add
    // your own for even more awesome. Any Psy command added here will be
    // available in your Psy shell sessions.
    'commands' => array(

        // The `parse` command is a command used in the development of PsySH.
        // Given a string of PHP code, it pretty-prints the
        // [PHP Parser](https://github.com/nikic/PHP-Parser) parse tree. It
        // prolly won't be super useful for most of you, but it's there if you
        // want to play :)
        new \Psy\Command\ParseCommand,
    ),

    // PsySH uses symfony/var-dumper's casters for presenting scalars, resources,
    // arrays and objects. You can enable additional casters, or write your own!
    // See http://symfony.com/doc/current/components/var_dumper/advanced.html#casters
    'casters' => array(
        'MyFooClass' => 'MyFooClassCaster::castMyFooObject',
    ),

    // You can disable tab completion if you want to. Not sure why you'd want to.
    'tabCompletion' => false,

    // You can write your own tab completion matchers, too! Here are some that enable
    // tab completion for MongoDB database and collection names:
    'tabCompletionMatchers' => array(
        new \Psy\TabCompletion\Matcher\MongoClientMatcher,
        new \Psy\TabCompletion\Matcher\MongoDatabaseMatcher,
    ),

    // If multiple versions of the same configuration or data file exist, PsySH will
    // use the file with highest precedence, and will silently ignore all others. With
    // this enabled, a warning will be emitted (but not an exception thrown) if multiple
    // configuration or data files are found.
    //
    // This will default to true in a future release, but is false for now.
    'warnOnMultipleConfigs' => true,
);
?>
```