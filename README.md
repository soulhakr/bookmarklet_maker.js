bookmarklet_maker.js
====================

A Node.js-compatible version of the [JavaScript Bookmarklet Builder](http://daringfireball.net/2007/03/javascript_bookmarklet_builder) utility from John Gruber, because… why use PERL to build JavaScript?

Introduction
------------

From the Gruber's site:

> …a bookmarklet is a little JavaScript script that’s intended to be run from a web browser’s bookmarks bar or menu. The reason they work as “bookmarks” is that the JavaScript source code is crammed into the form of a URL using the “javascript:” scheme… Developing or modifying bookmarklets can be irritating, to say the least, because of this requirement that the JavaScript code be in the form of a URL.

So their solution was to provide a PERL script which "compiles" the javascript into a UTF8-encoded `javascript:` URI:


    #!/usr/bin/env perl
    #
    # http://daringfireball.net/2007/03/javascript_bookmarklet_builder
    # Licence: http://www.opensource.org/licenses/mit-license.php
    
    use strict;
    use warnings;
    use URI::Escape qw(uri_escape_utf8);
    use open  IO  => ":utf8",       # UTF8 by default
              ":std";               # Apply to STDIN/STDOUT/STDERR
    
    my $src = do { local $/; <> };
    
    # Zap the first line if there's already a bookmarklet comment:
    $src =~ s{^// ?javascript:.+\n}{};
    my $bookmarklet = $src;
    
    for ($bookmarklet) {
        s{^\s*//.+\n}{}gm;  # Kill comments.
        s{\t}{ }gm;         # Tabs to spaces
        s{[ ]{2,}}{ }gm;    # Space runs to one space
        s{^\s+}{}gm;        # Kill line-leading whitespace
        s{\s+$}{}gm;        # Kill line-ending whitespace
        s{\n}{}gm;          # Kill newlines
    }
    
    # Escape single- and double-quotes, spaces, control chars, unicode:
    $bookmarklet = "javascript:" .
        uri_escape_utf8($bookmarklet, qq('" \x00-\x1f\x7f-\xff));
    
    print "// $bookmarklet\n" . $src;
    
    # Put bookmarklet on clipboard:
    `/bin/echo -n '$bookmarklet' | /usr/bin/pbcopy`;

Objectives
----------

The above implementation has some problems which this implementation seeks to correct:
  - The default behavior in some (most?) browsers for handling the `javascript:` URI schema is to redirect the navigator to a new page displaying the first uncaught return value of any statements in the URI outside of function scope. Bookmarklets should typecast the code in a `void()` or encapsulated in the familiar self-invoking function (i.e. `(function () { var foo, etc… })();`) to avoid this.
  - Without scoping, your bookmarklets may clobber variables in the global namespace
  - I don't know about others, but I generally prefer working the same language if at all practical. If I'm working in JavaScript, I'd rather shift gears to PERL as little as possible
  - As bookmarklets grow in complexity, one should optimally be able to run the code through minification before deployment 
  - One should be able to write and debug a bookmarklet in `"use strict";` mode, and run the code through [jsLint](http://jslint.org) or somesuch to aid in debugging and maintainability.
