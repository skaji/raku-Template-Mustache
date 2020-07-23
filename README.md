[![Build Status](https://travis-ci.org/softmoth/raku-Template-Mustache.svg?branch=master)](https://travis-ci.org/softmoth/raku-Template-Mustache) [![Windows Status](https://ci.appveyor.com/api/projects/status/github/softmoth/raku-Template-Mustache?branch=master&passingText=Windows%20-%20OK&failingText=Windows%20-%20FAIL&pendingText=Windows%20-%20pending&svg=true)](https://ci.appveyor.com/project/softmoth/raku-Template-Mustache/branch/master)

Raku implementation of Mustache templates, [http://mustache.github.io/](http://mustache.github.io/).

Synopsis
========

```raku
use Template::Mustache;

# Call .render as a class method
Template::Mustache.render('Hello, {{planet}}!', { planet => 'world' }).say;

# Or instantiate an instance
my $stache = Template::Mustache.new: :from<./views>;

# Subroutines are called
say $stache.render('The time is {{time}}', {
    time => { ~DateTime.new(now).local }
});

my @people =
    { :name('James T. Kirk'), :title<Captain> },
    { :name('Wesley'), :title('Dread Pirate'), :emcee },
    { :name('Dana Scully'), :title('Special Agent') },
    ;

# See this template in ./t/views/roster.mustache
$stache.render('roster', { :@people }).say;

my %context =
    event => 'Masters of the Universe Convention',
    :@people,
    ;
my %partials =
    welcome =>
        qq:b{Welcome to the {{event}}! We’re pleased to have you here.\n\n},
    ;

# See this result in ./t/50-readme.t
Template::Mustache.render(q:to/EOF/,
        {{> welcome}}
        {{> roster}}

            Dinner at 7PM in the Grand Ballroom. Bring a chair!
        EOF
    %context,
    :from([%partials, './views'])
).say;
```

Description
===========

Logging
-------

### Log levels

Messages are logged with varying severity levels (from most to least severe): `Fatal`, `Error`, `Warn`, `Info`, `Verbose`, `Audit`, `Debug`, `Trace`, `Trace2`

By default, only messages of `Error` or worse are logged. That default can be changed with the `TEMPLATE_MUSTACHE_LOGLEVEL` environment variable.

    TEMPLATE_MUSTACHE_LOGLEVEL=Debug

The default is overridden with the `:log-level` option to `Template::Mustache.new`, or a `Template::Mustache::Logger` object can be passed via the `:logger` option.

```raku
my $stache = Template::Mustache.new: :log-level<Trace>;

my $logger = Template::Mustache::Logger.new: :level<Debug>;
my $stache = Template::Mustache.new: :$logger;
```

Either method can be used with the `.render` method, as well.

```raku
my %data = hello => 'world';

Template::Mustache.render: 'Hello, {{hello}}!', %data, :log-level<Trace>;

my $logger = Template::Mustache::Logger.new: :level<Debug>;
Template::Mustache.render: 'Hello, {{hello}}!', %data, :$logger;
```

### Log routine

By default, any messages at level `Warn` or worse are logged with the `warn` routine. A `CONTROL` block can handle such warnings if needed; see [Language/phasers](https://docs.raku.org/language/phasers#CONTROL) for details. Less severe messages (`Info` and up) are logged with the `note` routine.

The routine can be set per log level, in the `Template::Mustache::Logger.routines` hash.

```raku
# Use say instead of note for Info and up; the more severe
# levels (C<Warn> down to C<Fatal>) still use the warn routine
my $stache = Template::Mustache.new: :log-routine(&say);

# But even those can be set explicitly
$stache.logger.routines{$_} = &die for <Warn Error Fatal>;

$stache.render: '{{missing}}', {};  # dies
```

### method log

  * `multi method log(Exception $exception, LogLevel :$level)`

  * `multi method log(LogLevel :$level, *@msgs)`

Emit a message at `$level` (`Info` by default).

Extra features
==============

Template inheritence
--------------------

Support for `hogan.js`-style [template inheritence](https://github.com/groue/GRMustache/blob/master/Guides/template_inheritance.md) is available.

Pragma: KEEP-UNUSED-VARIABLES
-----------------------------

Specify `:pragma<KEEP-UNUSED-VARIABLES>` to either `Template::Mustache.new` or `.render`, and any variables which are not defined in the data context will be kept in the rendered text. See `t/13-pragmas.t` for examples.

More Examples and Tests
=======================

The Mustache spec provides a wealth of examples to demonstrate exactly how the format behaves.

[https://github.com/mustache/spec/tree/master/specs/](https://github.com/mustache/spec/tree/master/specs/)

All of the official Mustache spec tests pass. An updated copy of the tests is distributed in `t/specs`.

To check against the official (outdated) specs repository, clone it into `../mustache-spec`: [https://travis-ci.org/softmoth/raku-Template-Mustache](https://travis-ci.org/softmoth/raku-Template-Mustache). The `perl6` branch just updates the .json files to match the .yml sources (needed until someone writes a compliant YAML parser in Raku … hint, hint), and adds Raku lambda code strings for that portion of the specs.

    # Ensure you are using the default 'perl6' branch, not 'master'
    git clone git@github.com:softmoth/mustache-spec.git ../mustache-spec
    git branch -v

The test file `t/specs/inheritable_partials.json` is taken from [groue/GRMustache](https:/github.com/groue/GRMustache).

Other Mustache Implementations
==============================

There are many, many Mustache implementations in various languages. Some of note are:

  * The original Ruby version [https://github.com/defunkt/mustache](https://github.com/defunkt/mustache)

  * Twitter's hogan.js [https://github.com/twitter/hogan.js](https://github.com/twitter/hogan.js)

  * mustache.java [https://github.com/spullara/mustache.java](https://github.com/spullara/mustache.java)

  * GRMustache (Objective C) [https://github.com/groue/GRMustache](https://github.com/groue/GRMustache)

  * mustache.php [https://github.com/bobthecow/mustache.php](https://github.com/bobthecow/mustache.php)

TODO
====

  * full object support (with method calls; currently the object is just stringified)

  * global helpers (context items that float at the top of the stack)

  * database loader

  * pragmas (FILTERS?)

License
=======

[Artistic License 2.0](http://www.perlfoundation.org/artistic_license_2_0)

