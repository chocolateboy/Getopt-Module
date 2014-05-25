## Getopt::Module

- [NAME](#name)
- [SYNOPSIS](#synopsis)
- [DESCRIPTION](#description)
- [EXPORTS](#exports)
    - [GetModule](#getmodule)
    - [TARGETS](#targets)
        - [ArrayRef](#arrayref)
        - [CodeRef](#coderef)
        - [HashRef](#hashref)
        - [ScalarRef](#scalarref)
    - [OPTIONS](#options)
        - [no\_import](#no\_import)
        - [separator](#separator)
- [VERSION](#version)
- [SEE ALSO](#see-also)
- [AUTHOR](#author)
- [COPYRIGHT AND LICENSE](#copyright-and-license)

## NAME

Getopt::Module - handle -M and -m options like perl

## SYNOPSIS

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

```bash

    command -Mautobox::Core -MBar=baz,quux -e '$_->split(...)->map(...)->join(...)'

```

## DESCRIPTION

This module provides a convenient way for command-line Perl scripts to handle `-M`
and `-m` options in the same way as perl.

## EXPORTS

None by default.

### GetModule

__Signature__: (ArrayRef | CodeRef | HashRef | ScalarRef \[, Hash | HashRef \]) -> CodeRef

    my $sub = GetModule($target, %options);

Takes a target and an optional hash or hashref of [options](#OPTIONS) and returns a subroutine that can be used
to handle a [Getopt::Long](http://search.cpan.org/perldoc?Getopt::Long) option. The option's value is parsed and its components (module name,
import type and parameters) are assigned to the target in the following ways.

#### TARGETS

##### ScalarRef

`eval`able `use`/`no` statements are appended to the referenced scalar, separated by the ["separator"](#separator) option.
If no separator is supplied, it defaults to a single space (" ") e.g.:

Command:

    command -MFoo=bar -M-Baz=quux

Usage:

    my $statements;

    GetOptions(
        'M|module=s' => GetModule(\$statements),
    );

Result (`$statements`):

    "use Foo qw(bar); no Baz qw(quux);"

##### ArrayRef

The `use`/`no` statement is pushed onto the arrayref e.g.:

Command:

    command -MFoo=bar,baz -M-Quux

Usage:

    my $modules = [];

    GetOptions(
        'M|module=s' => GetModule($modules),
    );

Result (`$modules`):

    [ "use Foo qw(bar baz);", "no Quux;" ]

##### HashRef

Pushes the statement onto the arrayref pointed to by `$hash->{ $module_name }`, creating it if it doesn't exist. e.g.:

Command:

    command -MFoo=bar -M-Foo=baz -MQuux

Usage:

    my $modules = {};

    GetOptions(
        'M|module=s' => GetModule($modules);
    );

Result (`$modules`):

    {
        Foo  => [ "use Foo qw(bar);", "no Foo qw(baz);" ],
        Quux => [ "use Quux;" ],
    }

##### CodeRef

The coderef is passed 3 parameters:

- name

    The name of the [Getopt::Long](http://search.cpan.org/perldoc?Getopt::Long) option e.g. `M`.

- eval

    The option's value as a `use` or `no` statement e.g: "use Foo qw(bar baz);".

- spec

    A hashref that makes the various components of the option available e.g.:

    Command:

        command -MFoo=bar,baz

    Usage:

        sub process_module { ... }

        GetOptions(
            'M|module=s' => GetModule(\&process_module);
        );

    The following hashref would be passed as the third argument to the `process_module` sub:

        {
            args      => 'bar,baz',              # the supplied import/unimport args; undef if none are supplied
            eval      => 'use Foo qw(bar baz);', # the evalable statement representing the option's value
            method    => 'import',               # the method call represented by the statement: either "import" or "unimport"
            module    => 'Foo'                   # the module name
            name      => 'M',                    # the Getopt::Long option name
            statement => 'use',                  # the statement type: either "use" or "no"
            value     => 'Foo=bar,baz',          # The Getopt::Long option value
        }

#### OPTIONS

##### no\_import

By default, if no `import`/`unimport` parameters are supplied e.g.:

    command -MFoo

the `use`/`no` statement omits the parameter list:

    use Foo;

If no parameters are supplied and `no_import` is set to a true value, the resulting statement disables the
`import`/`unimport` method call by passing an empty list e.g:

    use Foo ();

This corresponds to perl's `-m` option.

##### separator

The separator used to separate statements assigned to the scalar-ref target. Default: a single space (" ").

## VERSION

0.0.2

## SEE ALSO

- [Getopt::ArgvFile](http://search.cpan.org/perldoc?Getopt::ArgvFile)
- [Getopt::Long](http://search.cpan.org/perldoc?Getopt::Long)
- [perlrun](http://search.cpan.org/perldoc?perlrun)

## AUTHOR

chocolateboy <chocolate@cpan.org>

## COPYRIGHT AND LICENSE

Copyright (C) 2014 by chocolateboy

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself, either Perl version 5.14.2 or,
at your option, any later version of Perl 5 you may have available.
