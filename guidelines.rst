Porting Guidelines
==================

Here some basic patterns to follow:

- Python Versions: 2.6 or higher and 3.3 and higher
- String Literals: u'' for unicode, b'' for bytes, '' for version specific auto-switch
- No 2to3!
- Start with tests, verify them, then code.
