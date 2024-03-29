# NAME

Mojo::Promisify - Convert callback code to promise based code

# SYNOPSIS

    use Mojo::Promisify qw(promisify_call promisify_patch);
    use Some::NonBlockingClass;

    # Create an object from a callback based class
    my $nb_obj = Some::NonBlockingClass->new;

    # Call a callback based method, but return a Mojo::Promise
    promisify_call($nb_obj => get_stuff_by_id => 42)
      ->then(sub { print @_ })
      ->catch(sub { warn $_[0] });

    # Add a method that wraps around the callback based method and return a
    # Mojo::Promise.
    promisify_patch "Some::NonBlockingClass" => "get_stuff_by_id";

    # The added method has the "_p" suffix
    $nb_obj->get_stuff_by_id_p(42)->then(sub { print @_ });

# DESCRIPTION

[Mojo::Promisify](https://metacpan.org/pod/Mojo::Promisify) is a utility module that can upgrade your legacy callback
based API to a [Mojo::Promise](https://metacpan.org/pod/Mojo::Promise) based API.

It might not be the most efficient way to run your code, but it will allow
you to easily add methods that will return promises.

This module only works with methods that passes on `$err` as the first argument
to the callback, like this:

    sub get_stuff_by_id {
      my ($self, $id, $cb) = @_;

      my $err = "Some error";
      my $res = undef;
      Mojo::IOLoop->next_tick(sub { $self->$cb($err, $res) });

      return $self;
    }

The wrapped method can however pass on as many arguments as it wants after the
`$err` and all will be passed on to the fulfillment callback in the promise.
The promise will be rejected if `$err` has a value.

Note that this module is currently EXPERIMENTAL, but it will most probably not
change much.

# FUNCTIONS

## promisify

    $code = promisify($obj => $method);
    $promise = $code->(@args);

Returns a closure that wraps around a given `$method` for the `$obj` and
returns a promise. `@args` are the same arguments you would normally give to
the `$method`, but without the callback at the end.

## promisify\_call

    $promise = promisify_call($obj => $method, @args);

This function is the same as:

    $promise = promisify($obj => $method)->(@args);

## promisify\_patch

    promisify_patch $class, @methods;

Used to patch a class with new promise based methods. The methods that are
patched in will have the "\_p" suffix added.

Note that this function _will_ replace existing methods!

# AUTHOR

Jan Henning Thorsen

# COPYRIGHT AND LICENSE

Copyright (C) 2019, Jan Henning Thorsen.

This program is free software, you can redistribute it and/or modify it under
the terms of the Artistic License version 2.0.

# SEE ALSO

[Mojo::Promise](https://metacpan.org/pod/Mojo::Promise)
