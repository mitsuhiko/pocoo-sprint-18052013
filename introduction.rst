Werkzeug / Flask / Jinja2 Online Hackaton 18th of May 2013
==========================================================

Hey everybody.  While this was supposed to be an offline and online
experience I did not manage to get a space organized that would work out
for the 18th which will make this an online only event.

There are two immediate goals for it:

1.  Redo the Jinja2 port to not use 2to3.  This is of high interest
    because currently one of the reasons not much changes in Jinja2 is
    that I'm afraid I break the Python 3 port.
2.  Port most of Werkzeug to 3.  There are already some patches in Ronny's
    repository and we should be able to build at least a basic version
    that uses that.  This will also mean some minor API changes for
    Werkzeug on Python 2 due to how some of the interface do not survive
    Python 3 otherwise.

When and Where?
---------------

The coordination for this will happen in the ``#pocoo-sprint`` IRC
channel on `irc.freenode.net`.  I'm in there with the `mitsuhiko` nickname
as you would expect.  To join all you need is an IRC client and connect to
that channel.

If you want to join join the IRC channel and add yourself to
particpants.rst in this repository and send a pull request.

What do I need to Bring?
------------------------

Not much really, everybody is welcome to join in.  It's online online this
time around so don't be shy.  Idling is allowed as well :)

If you don't think you're up for it still join, you can also help out just
going through pull requests and documentation.

In any case git experience is a very good thing to have, ideally you also
have a github account to make sharing things easy.

What's the Plan?
----------------

I will lay out a list of tickets and tag them appropriately for how we're
going to tackle things.  Generally it depends on the project, here is the
plan:

Werkzeug:

-   Ronny's port generally is good enough as a starting point.  We will
    however need to verify that all the API did not take a huge hit and
    performs as it should do.
    
    We will start with looking at the port and verify testcase for
    testcase if it makes sense.
-   The Werkzeug documentation is pretty bad in general, some cleanup
    there will be necessary and a new chapter on Python 3 is required.

Jinja2:

-   This one is tricky and will probably be a two step approach.  Jinja2
    currently uses 2to3 for the Python 3 support and it's pretty ugly.
    The idea would be to use python-modernize, a version of 2to3 that
    spits out code that works with 2.x and 3.x.

    I did some tests on it recently and there are some problems with it.
    Maybe it would make sense to try to improve python-modernize before we
    tackle that thing.

Flask:

-   Step one would be to go through the codebase with python-modernize and
    fix the issues that it leaves behind.  Ideally we have a codebase that
    would import by itself on Python 2 and Python 3.  The code is already
    written in a way that it does not require much porting.
-   In theory when we do that we should once Werkzeug catches up have a
    version of Flask that runs on Python 3

What about an Offline Sprint?
-----------------------------

I am aiming for June 1st to have an offline sprint in London if people are
interested in it.  There will be stuff remaining after Saturday and it
only makes sense to have an offline one.  If that happens I will sponsor
some pizzas and beer for the cause :-)
