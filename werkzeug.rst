Sprint notes: https://gist.github.com/mitsuhiko/5583127

General
=======

This text aims to document the ***official*** ongoing process of porting Werkzeug_ to Python 3. The primary repository for this is located at `RonnyPfannschmidt/werkzeug`_, the following branches are relevant to the port:

`python3-urllib` contains the most recent commits.

The port is supposed to only support Py3.3+ and Py2.7+ for now, so we can use ``u""`` and ``b""`` for literals for both 2 and 3. See `PEP 414`_.

.. _Werkzeug: http://werkzeug.pocoo.org/
.. _RonnyPfannschmidt/werkzeug: https://github.com/RonnyPfannschmidt/werkzeug
.. _PEP 414: http://www.python.org/dev/peps/pep-0414/


Why another one? There are already finished ones out there!
-----------------------------------------------------------

It's hard to argue against something that is already working in practice. Yet we already found some parts of Werkzeug which should, in our opinion, be rewritten more radically (with API breakage), in order to adapt to the changes in Python 3. That surely means this port is going to take longer to finish, but that doesn't bother us.

A validation of this idea is that Jinja will have to be ported for a second time, because 2to3 turned out to be an impractical solution. That's why we think it'd be a good idea to get it right the first time.


gotchas
=======

- Just "casting" a bytestring to unicode with ``six.text_type(...)`` won't work.

  >>> str(b'foobar')  # gives us the repr of the bytestring!
  "b'foobar'"

- The ``bytes`` type in Python 3 behaves completely different than the ``bytes`` (`` = str``) in Python 2.

  For example, iteration:

  >>> list(b'foobar')  # Python 2
  ['f', 'o', 'o', 'b', 'a', 'r']

  >>> list(b'foobar')  # Python 3
  [102, 111, 111, 98, 97, 114]

  In ``werkzeug._urlparse`` there's an undocumented function called ``_iter_bytestring``, if it turns out to be needed in other modules feel free to move it to ``_compat``.

Submodules and -packages
========================

_compat.py
----------

This module contains common Python 3 helperfunctions and -classes that are not included in six, as well as many try-except statements for importing. It's only sensible to move something into ``_compat`` if is used by more than one module.

datastructures.py
-----------------

- To create an iterating API similar to the native dicts of Py2 and Py3 (for ``MultiDict`` and friends) there's a class decorator called ``native_itermethods``. See https://gist.github.com/untitaker/5389383 for an example.

urls.py
-------

``werkzeug.urls`` provides several wrapper functions for Python 2 urlparse (such as ``_(un)quote``, ``safe_urlsplit``), whose main purpose is to work around the behavior of the Py2 stdlib and its lack of unicode support. While this is already a somewhat inconvenient situation, it gets even more complicated because the Py3 ``urllib.parse`` actually does handle unicode properly. In other words, the mentioned wrapper functions are now wrapping two libraries with completely different behavior.

The easiest way out of this is to backport Py3's ``urllib.parse`` to run on both Py2 and Py3, and bundle it with Werkzeug, the fixes can then be directly merged into there.

The problem is that the behavior between Werkzeug's wrapper functions and ``urllib.parse`` seem to differ in subtle and hard to spot ways, which still have fatal consequences. At least that was my (untitaker) impression when i tinkered around with it. @RonnyPfannschmidt said there surely is "something obvious we're missing". Probably a re-thinking on how URLs and URIs are represented internally is appropriate at this point.

Pending Changes
---------------

These are things that need to be done:

Headers
````````

1.  Headers and EnvironHeaders are internally now always dealing with
    unicode.  The strings they contain are strictly limited to latin1
    but only on output is this checked.
2.  We remove `to_list` and add `to_wsgi_list` that takes no arguments
    and returns unicode on 3.x for the server to encode and on 2.x we
    do the encode in the datastructure.
3.  Remove `Headers.linked`, instead have a subclass of `Headers` that
    is a `LinkedHeaders` which does the encode/decode as necessary on
    every single access.

Keys
````

-   keys in dictionaries are currently usually bytestrings in 2.x and
    sometimes unicode.
-   With the new rule it's:

    -   native strings on 3.x
    -   native strings in 2.x if ascii only
    -   unicode strings in 2.x if non ascii

URIs / IRIs
```````````

1.  All URIs are represented as native string in their canonical form.
2.  All IRIs are always represented as unicode strings.
3.  All functions also accept bytes on input (both IRI and URI)
4.  URI->IRI->IRI->IRI needs to be idempotent
5.  we need to add URI/IRI parsing and manipulation functions that work
    regardless of the protocol and string type and return URI/IRI objects
    (to replace the ill fated urlparse.urlparse / urlparse.urljoin)

Stream Handlers
```````````````

All stream objects should support unicode and bytes.  If auto detection is
impossible we need to have different classes for them (StringStream and
BytesStream etc.).
