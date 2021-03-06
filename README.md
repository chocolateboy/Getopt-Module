# Getopt::Module

[![Build Status](https://github.com/chocolateboy/Getopt-Module/workflows/test/badge.svg)](https://github.com/chocolateboy/Getopt-Module/actions?query=workflow%3Atest)
[![CPAN Version](https://badge.fury.io/pl/Getopt-Module.svg)](https://badge.fury.io/pl/Getopt-Module)

<!-- TOC -->

- [NAME](#name)
- [SYNOPSIS](#synopsis)
- [DESCRIPTION](#description)
- [EXPORTS](#exports)
  - [GetModule](#getmodule)
    - [TARGETS](#targets)
      - [ScalarRef](#scalarref)
      - [ArrayRef](#arrayref)
      - [HashRef](#hashref)
      - [CodeRef](#coderef)
    - [OPTIONS](#options)
      - [no_import](#no_import)
      - [separator](#separator)
- [VERSION](#version)
- [SEE ALSO](#see-also)
- [AUTHOR](#author)
- [COPYRIGHT AND LICENSE](#copyright-and-license)

<!-- TOC END -->

# NAME

Getopt::Module - handle -M and -m options like perl

# SYNOPSIS

```perl
use Getopt::Long;
use Getopt::Module qw(GetModule);

my ($modules, $eval);

GetOptions(
    'M|module=s' => GetModule(\$modules),
    'm=s'        => GetModule(\$modules, no_import => 1),
    'e|eval=s'   => \$eval,
);

my $sub = eval "sub { $modules $eval }";
```

    $ command -Mautobox::Core -MBar=baz,quux -e '$_->split(...)->map(...)->join(...)'

# DESCRIPTION

This module provides a convenient way for command-line Perl scripts to handle
`-M` and `-m` options in the same way as perl.

# EXPORTS

None by default.

## GetModule

**Signature**: `(ArrayRef | CodeRef | HashRef | ScalarRef [, Hash | HashRef ]) → (Str, Str) → HashRef`

```perl
my $sub = GetModule($target, %options);
```

Takes a target and an optional hash or hashref of [options](#options) and
returns a
[subroutine](https://metacpan.org/pod/Getopt::Long#User-defined-subroutines-to-handle-options)
which takes an option name and a perl `-M`/`-m`-style option value and assigns
the value's components (module name, import type and parameters) to the target
in the following ways.

### TARGETS

#### ScalarRef

`eval`able `use`/`no` statements are appended to the referenced scalar,
separated by the ["separator"](#separator) option. If no separator is supplied,
it defaults to a single space (" "), e.g.:

Command:

    command -MFoo=bar -M-Baz=quux

Usage:

```perl
my $modules;

GetOptions(
    'M|module=s' => GetModule(\$modules),
);
```

Result (`$modules`):

```perl
"use Foo qw(bar); no Baz qw(quux);"
```

#### ArrayRef

The `use`/`no` statement is pushed onto the arrayref, e.g.:

Command:

    command -MFoo=bar,baz -M-Quux

Usage:

```perl
my $modules = [];

GetOptions(
    'M|module=s' => GetModule($modules),
);
```

Result (`$modules`):

```perl
[ "use Foo qw(bar baz);", "no Quux;" ]
```

#### HashRef

Pushes the statement onto the arrayref pointed to by `$hash->{ $module_name }`,
creating it if it doesn't exist, e.g.:

Command:

    command -MFoo=bar -M-Foo=baz -MQuux

Usage:

```perl
my $modules = {};

GetOptions(
    'M|module=s' => GetModule($modules),
);
```

Result (`$modules`):

```perl
{
    Foo  => [ "use Foo qw(bar);", "no Foo qw(baz);" ],
    Quux => [ "use Quux;" ],
}
```

#### CodeRef

The coderef is passed 3 arguments:

<!-- TOC:ignore -->
##### name

The name of the [Getopt::Long](https://metacpan.org/pod/Getopt::Long) option, e.g. `M`.

<!-- TOC:ignore -->
##### eval

The option's value as a `use` or `no` statement, e.g.: "use Foo qw(bar baz);".

<!-- TOC:ignore -->
##### parsed

A hashref containing the option's parsed components, e.g.:

Command:

    command -MFoo=bar,baz

Usage:

```perl
sub callback { ... }

GetOptions(
    'M|module=s' => GetModule(\&callback),
);
```

The following hashref would be passed as the third argument to the `callback` sub:

```perl
{
    args      => 'bar,baz',              # the import/unimport args; undef if none are supplied
    eval      => 'use Foo qw(bar baz);', # the evalable statement representing the option's value
    method    => 'import',               # the method call represented by the statement: either "import" or "unimport"
    module    => 'Foo'                   # the module name
    name      => 'M',                    # the Getopt::Long option name
    statement => 'use',                  # the statement type: either "use" or "no"
    value     => 'Foo=bar,baz',          # The Getopt::Long option value
}
```

### OPTIONS

#### no_import

By default, if no `import`/`unimport` parameters are supplied, e.g.:

    command -MFoo

the `use`/`no` statement omits the parameter list:

```perl
use Foo;
```

If no parameters are supplied and `no_import` is set to a true value, the
resulting statement disables the `import`/`unimport` method call by passing an
empty list, e.g.:

```perl
use Foo ();
```

This corresponds to perl's `-m` option.

#### separator

The separator used to separate statements assigned to the scalar-ref target.
Default: a single space (" ").

# VERSION

1.0.0

# SEE ALSO

- [Getopt::ArgvFile](https://metacpan.org/pod/Getopt::ArgvFile)
- [Getopt::Long](https://metacpan.org/pod/Getopt::Long)
- [perlrun](https://metacpan.org/pod/perlrun)

# AUTHOR

[chocolateboy](mailto:chocolate@cpan.org)

# COPYRIGHT AND LICENSE

Copyright © 2014-2021 by chocolateboy.

This is free software; you can redistribute it and/or modify it under the
terms of the [Artistic License 2.0](https://www.opensource.org/licenses/artistic-license-2.0.php).
