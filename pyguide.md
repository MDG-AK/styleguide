<!--
AUTHORS:
Prefer only GitHub-flavored Markdown in external text.
See README.md for details.
-->

# Inspired from Google Python Style Guide


<a id="background"></a>
## 1 Background

Python is the main dynamic language used at MDG. This style guide is a list
of *dos and don'ts* for Python programs. You must try to follow this guidelines
if possible.  

You can use any IDE of your choice. If you need help with formatting code
correctly, you can use the settings file created by Google [settings file for
Vim](google_python_style.vim). For Emacs, the default settings should be fine.

Many teams use the [yapf](https://github.com/google/yapf/)
auto-formatter to avoid arguing over formatting.


<a id="s2-python-language-rules"></a>
<a id="python-language-rules"></a>
## 2 Python Language Rules

<a id="s2.2-imports"></a>
<a id="imports"></a>
### 2.2 Imports

Use `import` statements for packages and modules only, not for individual
classes or functions. Note that there is an explicit exemption for imports from
the [typing module](#typing-imports).

<a id="s2.2.1-definition"></a>
#### 2.2.1 Definition

Reusability mechanism for sharing code from one module to another.

<a id="s2.2.2-pros"></a>
#### 2.2.2 Pros

The namespace management convention is simple. The source of each identifier is
indicated in a consistent way; `x.Obj` says that object `Obj` is defined in
module `x`.

<a id="s2.2.3-cons"></a>
#### 2.2.3 Cons

Module names can still collide. Some module names are inconveniently long.

<a id="s2.2.4-decision"></a>
#### 2.2.4 Decision

* Use `import x` for importing packages and modules.
* Use `from x import y` where `x` is the package prefix and `y` is the module
name with no prefix.
* Use `from x import y as z` if two modules named `y` are to be imported or if
`y` is an inconveniently long name.
* Use `import y as z` only when `z` is a standard abbreviation (e.g., `np` for
`numpy`).

For example the module `sound.effects.echo` may be imported as follows:

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

Do not use relative names in imports. Even if the module is in the same package,
use the full package name. This helps prevent unintentionally importing a
package twice.

Imports from the [typing module](#typing-imports) are exempt from this rule.

<a id="s2.3-packages"></a>
<a id="packages"></a>
### 2.3 Packages

Import each module using the full pathname location of the module.

<a id="s2.3.1-pros"></a>
#### 2.3.1 Pros

Avoids conflicts in module names or incorrect imports due to the module search
path not being what the author expected.  Makes it easier to find modules.

<a id="S2.3.2-cons"></a>
#### 2.3.2 Cons

Makes it harder to deploy code because you have to replicate the package
hierarchy.  Not really a problem with modern deployment mechanisms.

<a id="s2.3.3-decision"></a>
#### 2.3.3 Decision

All new code should import each module by its full package name.

Imports should be as follows:

Yes:

```python
# Reference absl.flags in code with the complete name (verbose).
import absl.flags
from doctor.who import jodie

FLAGS = absl.flags.FLAGS
```

```python
# Reference flags in code with just the module name (common).
from absl import flags
from doctor.who import jodie

FLAGS = flags.FLAGS
```

No: _(assume this file lives in `doctor/who/` where `jodie.py` also exists)_

```python
# Unclear what module the author wanted and what will be imported.  The actual
# import behavior depends on external factors controlling sys.path.
# Which possible jodie module did the author intend to import?
import jodie
```

The directory the main binary is located in should not be assumed to be in
`sys.path` despite that happening in some environments.  This being the case,
code should assume that `import jodie` refers to a third party or top level
package named `jodie`, not a local `jodie.py`.

<a id="s2.5-global-variables"></a>
<a id="global-variables"></a>
### 2.5 Global variables

Avoid global variables.

<a id="s2.5.1-definition"></a>
#### 2.5.1 Definition

Variables that are declared at the module level or as class attributes.

<a id="s2.5.2-pros"></a>
#### 2.5.2 Pros

Occasionally useful.

<a id="s2.5.3-cons"></a>
#### 2.5.3 Cons

Has the potential to change module behavior during the import, because
assignments to global variables are done when the module is first imported.

<a id="s2.5.4-decision"></a>
#### 2.5.4 Decision

Avoid global variables.

While they are technically variables, module-level constants are permitted and
encouraged. For example: `MAX_HOLY_HANDGRENADE_COUNT = 3`. Constants must be
named using all caps with underscores. See [Naming](#s3.16-naming) below.

If needed, globals should be declared at the module level and made internal to
the module by prepending an `_` to the name. External access must be done
through public module-level functions. See [Naming](#s3.16-naming) below.

<a id="s2.8-default-iterators-and-operators"></a>
<a id="default-iterators-and-operators"></a>
### 2.8 Default Iterators and Operators

Use default iterators and operators for types that support them, like lists,
dictionaries, and files.

<a id="s2.8.1-definition"></a>
#### 2.8.1 Definition

Container types, like dictionaries and lists, define default iterators and
membership test operators ("in" and "not in").

<a id="s2.8.2-pros"></a>
#### 2.8.2 Pros

The default iterators and operators are simple and efficient. They express the
operation directly, without extra method calls. A function that uses default
operators is generic. It can be used with any type that supports the operation.

<a id="s2.8.3-cons"></a>
#### 2.8.3 Cons

You can't tell the type of objects by reading the method names (e.g. has\_key()
means a dictionary). This is also an advantage.

<a id="s2.8.4-decision"></a>
#### 2.8.4 Decision

Use default iterators and operators for types that support them, like lists,
dictionaries, and files. The built-in types define iterator methods, too. Prefer
these methods to methods that return lists, except that you should not mutate a
container while iterating over it. Never use Python 2 specific iteration
methods such as `dict.iter*()` unless necessary.

```python
Yes:  for key in adict: ...
      if key not in adict: ...
      if obj in alist: ...
      for line in afile: ...
      for k, v in adict.items(): ...
      for k, v in six.iteritems(adict): ...
```

```python
No:   for key in adict.keys(): ...
      if not adict.has_key(key): ...
      for line in afile.readlines(): ...
      for k, v in dict.iteritems(): ...
```

<a id="s2.12-default-argument-values"></a>
<a id="default-argument-values"></a>
### 2.12 Default Argument Values

Okay in most cases.

<a id="s2.12.1-definition"></a>
#### 2.12.1 Definition

You can specify values for variables at the end of a function's parameter list,
e.g., `def foo(a, b=0):`.  If `foo` is called with only one argument,
`b` is set to 0. If it is called with two arguments, `b` has the value of the
second argument.

<a id="s2.12.2-pros"></a>
#### 2.12.2 Pros

Often you have a function that uses lots of default values, but-rarely-you want
to override the defaults. Default argument values provide an easy way to do
this, without having to define lots of functions for the rare exceptions. Also,
Python does not support overloaded methods/functions and default arguments are
an easy way of "faking" the overloading behavior.

<a id="s2.12.3-cons"></a>
#### 2.12.3 Cons

Default arguments are evaluated once at module load time. This may cause
problems if the argument is a mutable object such as a list or a dictionary. If
the function modifies the object (e.g., by appending an item to a list), the
default value is modified.

<a id="s2.12.4-decision"></a>
#### 2.12.4 Decision

Okay to use with the following caveat:

Do not use mutable objects as default values in the function or method
definition.

```python
Yes: def foo(a, b=None):
         if b is None:
             b = []
Yes: def foo(a, b: Optional[Sequence] = None):
         if b is None:
             b = []
Yes: def foo(a, b: Sequence = ()):  # Empty tuple OK since tuples are immutable
         ...
```

```python
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
         ...
```

<a id="s2.14-truefalse-evaluations"></a>
<a id="truefalse-evaluations"></a>
### 2.14 True/False evaluations

Use the "implicit" false if at all possible.

<a id="s2.14.1-definition"></a>
#### 2.14.1 Definition

Python evaluates certain values as `False` when in a boolean context. A quick
"rule of thumb" is that all "empty" values are considered false, so
`0, None, [], {}, ''` all evaluate as false in a boolean context.

<a id="s2.14.2-pros"></a>
#### 2.14.2 Pros

Conditions using Python booleans are easier to read and less error-prone. In
most cases, they're also faster.

<a id="s2.14.3-cons"></a>
#### 2.14.3 Cons

May look strange to C/C++ developers.

<a id="s2.14.4-decision"></a>
#### 2.14.4 Decision

Use the "implicit" false if at all possible, e.g., `if foo:` rather than
`if foo != []:`. There are a few caveats that you should keep in mind though:

-   Never use `==` or `!=` to compare singletons like `None`. Use `is` or
    `is not`.

-   Beware of writing `if x:` when you really mean `if x is not None:`-e.g.,
    when testing whether a variable or argument that defaults to `None` was set
    to some other value. The other value might be a value that's false in a
    boolean context!

-   Never compare a boolean variable to `False` using `==`. Use `if not x:`
    instead. If you need to distinguish `False` from `None` then chain the
    expressions, such as `if not x and x is not None:`.

-   For sequences (strings, lists, tuples), use the fact that empty sequences
    are false, so `if seq:` and `if not seq:` are preferable to `if len(seq):`
    and `if not len(seq):` respectively.

-   When handling integers, implicit false may involve more risk than benefit
    (i.e., accidentally handling `None` as 0). You may compare a value which is
    known to be an integer (and is not the result of `len()`) against the
    integer 0.

    ```python
    Yes: if not users:
             print('no users')

         if foo == 0:
             self.handle_zero()

         if i % 10 == 0:
             self.handle_multiple_of_ten()

         def f(x=None):
             if x is None:
                 x = []
    ```

    ```python
    No:  if len(users) == 0:
             print('no users')

         if foo is not None and not foo:
             self.handle_zero()

         if not i % 10:
             self.handle_multiple_of_ten()

         def f(x=None):
             x = x or []
    ```

-   Note that `'0'` (i.e., `0` as string) evaluates to true.



<a id="s3-python-style-rules"></a>
<a id="python-style-rules"></a>
## 3 Python Style Rules

<a id="s3.1-semicolons"></a>
<a id="semicolons"></a>
### 3.1 Semicolons

Do not terminate your lines with semicolons, and do not use semicolons to put
two statements on the same line.

<a id="s3.2-line-length"></a>
<a id="line-length"></a>
### 3.2 Line length

Maximum line length is *80 characters*.

Exceptions:

-   Long import statements.
-   URLs, pathnames, or long flags in comments.
-   Long string module level constants not containing whitespace that would be
    inconvenient to split across lines such as URLs or pathnames.
-   Pylint disable comments. (e.g.: `# pylint: disable=invalid-name`)

Do not use backslash line continuation except for `with` statements requiring
three or more context managers.

Make use of Python's [implicit line joining inside parentheses, brackets and
braces](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining).
If necessary, you can add an extra pair of parentheses around an expression.

```python
Yes: foo_bar(self, width, height, color='black', design=None, x='foo',
             emphasis=None, highlight=0)

     if (width == 0 and height == 0 and
         color == 'red' and emphasis == 'strong'):
```

When a literal string won't fit on a single line, use parentheses for implicit
line joining.

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

Within comments, put long URLs on their own line if necessary.

```python
Yes:  # See details at
      # http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

```python
No:  # See details at
     # http://www.example.com/us/developer/documentation/api/content/\
     # v2.0/csv_file_name_extension_full_specification.html
```

It is permissible to use backslash continuation when defining a `with` statement
whose expressions span three or more lines. For two lines of expressions, use a
nested `with` statement:

```python
Yes:  with very_long_first_expression_function() as spam, \
           very_long_second_expression_function() as beans, \
           third_thing() as eggs:
          place_order(eggs, beans, spam, beans)
```

```python
No:  with VeryLongFirstExpressionFunction() as spam, \
          VeryLongSecondExpressionFunction() as beans:
       PlaceOrder(eggs, beans, spam, beans)
```

```python
Yes:  with very_long_first_expression_function() as spam:
          with very_long_second_expression_function() as beans:
              place_order(beans, spam)
```

Make note of the indentation of the elements in the line continuation examples
above; see the [indentation](#s3.4-indentation) section for explanation.

<a id="s3.3-parentheses"></a>
<a id="parentheses"></a>
### 3.3 Parentheses

Use parentheses sparingly.

It is fine, though not required, to use parentheses around tuples. Do not use
them in return statements or conditional statements unless using parentheses for
implied line continuation or to indicate a tuple.

```python
Yes: if foo:
         bar()
     while x:
         x = bar()
     if x and y:
         bar()
     if not x:
         bar()
     # For a 1 item tuple the ()s are more visually obvious than the comma.
     onesie = (foo,)
     return foo
     return spam, beans
     return (spam, beans)
     for (x, y) in dict.items(): ...
```

```python
No:  if (x):
         bar()
     if not(x):
         bar()
     return (foo)
```


<a id="s3.4-indentation"></a>
<a id="indentation"></a>
### 3.4 Indentation

Indent your code blocks with *4 spaces*.

Never use tabs or mix tabs and spaces. In cases of implied line continuation,
you should align wrapped elements either vertically, as per the examples in the
[line length](#s3.2-line-length) section; or using a hanging indent of 4 spaces,
in which case there should be nothing after the open parenthesis or bracket on
the first line.

```python
Yes:   # Aligned with opening delimiter
       foo = long_function_name(var_one, var_two,
                                var_three, var_four)
       meal = (spam,
               beans)

       # Aligned with opening delimiter in a dictionary
       foo = {
           long_dictionary_key: value1 +
                                value2,
           ...
       }

       # 4-space hanging indent; nothing on first line
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four)
       meal = (
           spam,
           beans)

       # 4-space hanging indent in a dictionary
       foo = {
           long_dictionary_key:
               long_dictionary_value,
           ...
       }
```

```python
No:    # Stuff on first line forbidden
       foo = long_function_name(var_one, var_two,
           var_three, var_four)
       meal = (spam,
           beans)

       # 2-space hanging indent forbidden
       foo = long_function_name(
         var_one, var_two, var_three,
         var_four)

       # No hanging indent in a dictionary
       foo = {
           long_dictionary_key:
           long_dictionary_value,
           ...
       }
```

<a id="s3.4.1-trailing_comma"></a>
<a id="trailing_comma"></a>

### 3.4.1 Trailing commas in sequences of items?

Trailing commas in sequences of items are recommended only when the closing
container token `]`, `)`, or `}` does not appear on the same line as the final
element. The presence of a trailing comma is also used as a hint to our Python
code auto-formatter [YAPF](https://pypi.org/project/yapf/) to direct it to auto-format the container
of items to one item per line when the `,` after the final element is present.

```python
Yes:   golomb3 = [0, 1, 3]
Yes:   golomb4 = [
           0,
           1,
           4,
           6,
       ]
```

```python
No:    golomb4 = [
           0,
           1,
           4,
           6
       ]
```

<a id="s3.5-blank-lines"></a>
<a id="blank-lines"></a>
### 3.5 Blank Lines

Two blank lines between top-level definitions, be they function or class
definitions. One blank line between method definitions and between the `class`
line and the first method. No blank line following a `def` line. Use single
blank lines as you judge appropriate within functions or methods.

<a id="s3.6-whitespace"></a>
<a id="whitespace"></a>
### 3.6 Whitespace

Follow standard typographic rules for the use of spaces around punctuation.

No whitespace inside parentheses, brackets or braces.

```python
Yes: spam(ham[1], {eggs: 2}, [])
```

```python
No:  spam( ham[ 1 ], { eggs: 2 }, [ ] )
```

No whitespace before a comma, semicolon, or colon. Do use whitespace after a
comma, semicolon, or colon, except at the end of the line.

```python
Yes: if x == 4:
         print(x, y)
     x, y = y, x
```

```python
No:  if x == 4 :
         print(x , y)
     x , y = y , x
```

No whitespace before the open paren/bracket that starts an argument list,
indexing or slicing.

```python
Yes: spam(1)
```

```python
No:  spam (1)
```


```python
Yes: dict['key'] = list[index]
```

```python
No:  dict ['key'] = list [index]
```

Surround binary operators with a single space on either side for assignment
(`=`), comparisons (`==, <, >, !=, <>, <=, >=, in, not in, is, is not`), and
Booleans (`and, or, not`). Use your better judgment for the insertion of spaces
around arithmetic operators (`+`, `-`, `*`, `/`, `//`, `%`, `**`, `@`).

```python
Yes: x == 1
```

```python
No:  x<1
```

Never use spaces around `=` when passing keyword arguments or defining a default
parameter value, with one exception: [when a type annotation is
present](#typing-default-values), _do_ use spaces around the `=` for the default
parameter value.

```python
Yes: def complex(real, imag=0.0): return Magic(r=real, i=imag)
Yes: def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

```python
No:  def complex(real, imag = 0.0): return Magic(r = real, i = imag)
No:  def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

Don't use spaces to vertically align tokens on consecutive lines, since it
becomes a maintenance burden (applies to `:`, `#`, `=`, etc.):

```python
Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

```python
No:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }
```


<a id="s3.8-comments"></a>
<a id="comments"></a>
### 3.8 Comments and Docstrings

Be sure to use the right style for module, function, method docstrings and
inline comments.

<a id="s3.8.1-comments-in-doc-strings"></a>
<a id="comments-in-doc-strings"></a>
#### 3.8.1 Docstrings

Python uses _docstrings_ to document code. A docstring is a string that is the
first statement in a package, module, class or function. These strings can be
extracted automatically through the `__doc__` member of the object and are used
by `pydoc`.
(Try running `pydoc` on your module to see how it looks.) Always use the three
double-quote `"""` format for docstrings (per [PEP
257](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0257/)).
A docstring should be organized as a summary line (one physical line) terminated
by a period, question mark, or exclamation point, followed by a blank line,
followed by the rest of the docstring starting at the same cursor position as
the first quote of the first line. There are more formatting guidelines for
docstrings below.

<a id="s3.8.2-comments-in-modules"></a>
<a id="comments-in-modules"></a>
#### 3.8.2 Modules

Every file should contain license boilerplate. Choose the appropriate
boilerplate for the license used by the project (for example, Apache 2.0, BSD,
LGPL, GPL)

<a id="s3.8.3-functions-and-methods"></a>
<a id="functions-and-methods"></a>
#### 3.8.3 Functions and Methods

In this section, "function" means a method, function, or generator.

A function must have a docstring, unless it meets all of the following criteria:

-   not externally visible
-   very short
-   obvious

A docstring should give enough information to write a call to the function
without reading the function's code. The docstring should be descriptive
(`"""Fetches rows from a Bigtable."""`) rather than imperative
(`"""Fetch rows from a Bigtable."""`). A docstring should describe the
function's calling syntax and its semantics, not its implementation. For tricky
code, comments alongside the code are more appropriate than using docstrings.

A method that overrides a method from a base class may have a simple docstring
sending the reader to its overridden method's docstring, such as `"""See base
class."""`. The rationale is that there is no need to repeat in many places
documentation that is already present in the base method's docstring. However,
if the overriding method's behavior is substantially different from the
overridden method, or details need to be provided (e.g., documenting additional
side effects), a docstring with at least those differences is required on the
overriding method.

Certain aspects of a function should be documented in special sections, listed
below. Each section begins with a heading line, which ends with a colon.
Sections should be indented two spaces, except for the heading.

<a id="doc-function-args"></a>
[*Args:*](#doc-function-args)
:   List each parameter by name. A description should follow the name, and be
separated by a colon and a space. If the description is too long to fit on a
single 80-character line, use a hanging indent of 2 or 4 spaces (be
consistent with the rest of the file).<br>
The description should include required type(s) if the code does not contain
a corresponding type annotation.<br>
If a function accepts `*foo` (variable length argument lists) and/or `**bar`
(arbitrary keyword arguments), they should be listed as `*foo` and `**bar`.

<a id="doc-function-returns"></a>
[*Returns:* (or *Yields:* for generators)](#doc-function-returns)
:   Describe the type and semantics of the return value. If the function only
returns None, this section is not required. It may also be omitted if the
docstring starts with Returns or Yields (e.g.
`"""Returns row from Bigtable as a tuple of strings."""`) and the opening
sentence is sufficient to describe return value.

<a id="doc-function-raises"></a>
[*Raises:*](#doc-function-raises)
:   List all exceptions that are relevant to the interface.

```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
```

<a id="comments-in-block-and-inline"></a>
<a id="s3.8.5-comments-in-block-and-inline"></a>
#### 3.8.5 Block and Inline Comments

The final place to have comments is in tricky parts of the code. If you're going
to have to explain it at the next [code
review](http://en.wikipedia.org/wiki/Code_review), you should comment it
now. Complicated operations get a few lines of comments before the operations
commence. Non-obvious ones get comments at the end of the line.

```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
```

To improve legibility, these comments should be at least 2 spaces away from the
code.

On the other hand, never describe the code. Assume the person reading the code
knows Python (though not what you're trying to do) better than you do.

```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

<a id="s3.16-naming"></a>
<a id="naming"></a>
### 3.16 Naming

`module_name`,
`package_name`,
`ClassName`,
`method_name`,
`ExceptionName`,
`function_name`,
`GLOBAL_CONSTANT_NAME`,
`global_var_name`,
`instance_var_name`,
`function_parameter_name`,
`local_var_name`.

Function names, variable names, and filenames should be descriptive; eschew
abbreviation. In particular, do not use abbreviations that are ambiguous or
unfamiliar to readers outside your project, and do not abbreviate by deleting
letters within a word.

Always use a `.py` filename extension. Never use dashes.

<a id="s3.16.1-names-to-avoid"></a>
<a id="names-to-avoid"></a>
#### 3.16.1 Names to Avoid

-   single character names except for counters or iterators. You may use "e" as
    an exception identifier in try/except statements.
-   dashes (`-`) in any package/module name
-   `__double_leading_and_trailing_underscore__` names (reserved by Python)

<a id="s3.16.2-naming-conventions"></a>
<a id="naming-conventions"></a>
#### 3.16.2 Naming Convention

-   "Internal" means internal to a module, or protected or private within a
    class.

-   Prepending a single underscore (`_`) has some support for protecting module
    variables and functions (not included with `from module import *`). While
    prepending a double underscore (`__` aka "dunder") to an instance variable
    or method effectively makes the variable or method private to its class
    (using name mangling) we discourage its use as it impacts readability and
    testability and isn't *really* private.

-   Place related classes and top-level functions together in a
    module.
    Unlike Java, there is no need to limit yourself to one class per module.

-   Use CapWords for class names, but lower\_with\_under.py for module names.
    Although there are some old modules named CapWords.py, this is now
    discouraged because it's confusing when the module happens to be named after
    a class. ("wait -- did I write `import StringIO` or `from StringIO import
    StringIO`?")

-   Underscores may appear in *unittest* method names starting with `test` to
    separate logical components of the name, even if those components use
    CapWords. One possible pattern is `test<MethodUnderTest>_<state>`; for
    example `testPop_EmptyStack` is okay. There is no One Correct Way to name
    test methods.

<a id="s3.16.3-file-naming"></a>
<a id="file-naming"></a>
#### 3.16.3 File Naming

Python filenames must have a `.py` extension and must not contain dashes (`-`).
This allows them to be imported and unittested. If you want an executable to be
accessible without the extension, use a symbolic link or a simple bash wrapper
containing `exec "$0.py" "$@"`.

<a id="s3.16.4-guidelines-derived-from-guidos-recommendations"></a>
<a id="guidelines-derived-from-guidos-recommendations"></a>
#### 3.16.4 Guidelines derived from Guido's Recommendations

<table rules="all" border="1" summary="Guidelines from Guido's Recommendations"
       cellspacing="2" cellpadding="2">

  <tr>
    <th>Type</th>
    <th>Public</th>
    <th>Internal</th>
  </tr>

  <tr>
    <td>Packages</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Modules</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>Classes</td>
    <td><code>CapWords</code></td>
    <td><code>_CapWords</code></td>
  </tr>

  <tr>
    <td>Exceptions</td>
    <td><code>CapWords</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Functions</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code></td>
  </tr>

  <tr>
    <td>Global/Class Constants</td>
    <td><code>CAPS_WITH_UNDER</code></td>
    <td><code>_CAPS_WITH_UNDER</code></td>
  </tr>

  <tr>
    <td>Global/Class Variables</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>Instance Variables</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code> (protected)</td>
  </tr>

  <tr>
    <td>Method Names</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code> (protected)</td>
  </tr>

  <tr>
    <td>Function/Method Parameters</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Local Variables</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

</table>

While Python supports making things private by using a leading double underscore
`__` (aka. "dunder") prefix on a name, this is discouraged. Prefer the use of a
single underscore. They are easier to type, read, and to access from small
unittests. Lint warnings take care of invalid access to protected members.


<a id="s3.17-main"></a>
<a id="main"></a>
### 3.17 Main

Even a file meant to be used as an executable should be importable and a mere
import should not have the side effect of executing the program's main
functionality. The main functionality should be in a `main()` function.

In Python, `pydoc` as well as unit tests require modules to be importable. Your
code should always check `if __name__ == '__main__'` before executing your main
program so that the main program is not executed when the module is imported.

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

All code at the top level will be executed when the module is imported. Be
careful not to call functions, create objects, or perform other operations that
should not be executed when the file is being `pydoc`ed.

<a id="s3.18-function-length"></a>
<a id="function-length"></a>
### 3.18 Function length

Prefer small and focused functions.

We recognize that long functions are sometimes appropriate, so no hard limit is
placed on function length. If a function exceeds about 40 lines, think about
whether it can be broken up without harming the structure of the program.

Even if your long function works perfectly now, someone modifying it in a few
months may add new behavior. This could result in bugs that are hard to find.
Keeping your functions short and simple makes it easier for other people to read
and modify your code.

You could find long and complicated functions when working with
some code. Do not be intimidated by modifying existing code: if working with such
a function proves to be difficult, you find that errors are hard to debug, or
you want to use a piece of it in several different contexts, consider breaking
up the function into smaller and more manageable pieces.


## 4 Parting Words

*BE CONSISTENT*.

If you're editing code, take a few minutes to look at the code around you and
determine its style. If they use spaces around all their arithmetic operators,
you should too. If their comments have little boxes of hash marks around them,
make your comments have little boxes of hash marks around them too.

The point of having style guidelines is to have a common vocabulary of coding so
people can concentrate on what you're saying rather than on how you're saying
it. We present global style rules here so people know the vocabulary, but local
style is also important. If code you add to a file looks drastically different
from the existing code around it, it throws readers out of their rhythm when
they go to read it. Avoid this.
