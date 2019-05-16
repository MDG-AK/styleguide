<!--
AUTHORS:
Prefer only GitHub-flavored Markdown in external text.
See README.md for details.
-->

# Inspired from Google Python Style Guide


<a id="background"></a>
## 1 Background

Python is the main dynamic language used at MDG. This style guide is a list
of *dos and don'ts* for Python programs. You may follow this guidelines but you
are not required to do so.  

You can use any IDE of your choice. If you need help with formatting code
correctly, you can use the settings file created by Google [settings file for
Vim](google_python_style.vim). For Emacs, the default settings should be fine.

Many teams use the [yapf](https://github.com/google/yapf/)
auto-formatter to avoid arguing over formatting.


<a id="s2-python-language-rules"></a>
<a id="python-language-rules"></a>
## 2 Python Language Rules

<a id="s2.4-exceptions"></a>
<a id="exceptions"></a>
### 2.4 Exceptions

Exceptions are allowed but must be used carefully.

<a id="s2.4.1-definition"></a>
#### 2.4.1 Definition

Exceptions are a means of breaking out of the normal flow of control of a code
block to handle errors or other exceptional conditions.

<a id="s2.4.2-pros"></a>
#### 2.4.2 Pros

The control flow of normal operation code is not cluttered by error-handling
code. It also allows the control flow to skip multiple frames when a certain
condition occurs, e.g., returning from N nested functions in one step instead of
having to carry-through error codes.

<a id="s2.4.3-cons"></a>
#### 2.4.3 Cons

May cause the control flow to be confusing. Easy to miss error cases when making
library calls.

<a id="s2.4.4-decision"></a>
#### 2.4.4 Decision

Exceptions must follow certain conditions:

-   Raise exceptions like this: `raise MyError('Error message')` or `raise
    MyError()`. Do not use the two-argument form (`raise MyError, 'Error
    message'`).

-   Make use of built-in exception classes when it makes sense. For example,
    raise a `ValueError` if you were passed a negative number but were expecting
    a positive one. Do not use `assert` statements for validating argument
    values of a public API. `assert` is used to ensure internal correctness, not
    to enforce correct usage nor to indicate that some unexpected event
    occurred. If an exception is desired in the latter cases, use a raise
    statement. For example:


    ```python
    Yes:
      def connect_to_next_port(self, minimum):
        """Connects to the next available port.

        Args:
          minimum: A port value greater or equal to 1024.
        Raises:
          ValueError: If the minimum port specified is less than 1024.
          ConnectionError: If no available port is found.
        Returns:
          The new minimum port.
        """
        if minimum < 1024:
          raise ValueError('Minimum port must be at least 1024, not %d.' % (minimum,))
        port = self._find_next_open_port(minimum)
        if not port:
          raise ConnectionError('Could not connect to service on %d or higher.' % (minimum,))
        assert port >= minimum, 'Unexpected port %d when minimum was %d.' % (port, minimum)
        return port
    ```

    ```python
    No:
      def connect_to_next_port(self, minimum):
        """Connects to the next available port.

        Args:
          minimum: A port value greater or equal to 1024.
        Returns:
          The new minimum port.
        """
        assert minimum >= 1024, 'Minimum port must be at least 1024.'
        port = self._find_next_open_port(minimum)
        assert port is not None
        return port
    ```

-   Libraries or packages may define their own exceptions. When doing so they
    must inherit from an existing exception class. Exception names should end in
    `Error` and should not introduce stutter (`foo.FooError`).

-   Never use catch-all `except:` statements, or catch `Exception` or
    `StandardError`, unless you are re-raising the exception or in the outermost
    block in your thread (and printing an error message). Python is very
    tolerant in this regard and `except:` will really catch everything including
    misspelled names, sys.exit() calls, Ctrl+C interrupts, unittest failures and
    all kinds of other exceptions that you simply don't want to catch.

-   Minimize the amount of code in a `try`/`except` block. The larger the body
    of the `try`, the more likely that an exception will be raised by a line of
    code that you didn't expect to raise an exception. In those cases, the
    `try`/`except` block hides a real error.

-   Use the `finally` clause to execute code whether or not an exception is
    raised in the `try` block. This is often useful for cleanup, i.e., closing a
    file.

-   When capturing an exception, use `as` rather than a comma. For example:


    ```python
    try:
      raise Error()
    except Error as error:
      pass
    ```

<a id="list-comprehensions"></a>
<a id="s2.7-list_comprehensions"></a>
<a id="list_comprehensions"></a>
### 2.7 Comprehensions & Generator Expressions

Okay to use for simple cases.

<a id="s2.7.1-definition"></a>
#### 2.7.1 Definition

List, Dict, and Set comprehensions as well as generator expressions provide a
concise and efficient way to create container types and iterators without
resorting to the use of traditional loops, `map()`, `filter()`, or `lambda`.

<a id="s2.7.2-pros"></a>
#### 2.7.2 Pros

Simple comprehensions can be clearer and simpler than other dict, list, or set
creation techniques. Generator expressions can be very efficient, since they
avoid the creation of a list entirely.

<a id="s2.7.3-cons"></a>
#### 2.7.3 Cons

Complicated comprehensions or generator expressions can be hard to read.

<a id="s2.7.4-decision"></a>
#### 2.7.4 Decision

Okay to use for simple cases. Each portion must fit on one line: mapping
expression, `for` clause, filter expression. Multiple `for` clauses or filter
expressions are not permitted. Use loops instead when things get more
complicated.

```python
Yes:
  result = [mapping_expr for value in iterable if filter_expr]

  result = [{'key': value} for value in iterable
            if a_long_filter_expression(value)]

  result = [complicated_transform(x)
            for x in iterable if predicate(x)]

  descriptive_name = [
      transform({'key': key, 'value': value}, color='black')
      for key, value in generate_iterable(some_input)
      if complicated_condition_is_met(key, value)
  ]

  result = []
  for x in range(10):
      for y in range(5):
          if x * y > 10:
              result.append((x, y))

  return {x: complicated_transform(x)
          for x in long_generator_function(parameter)
          if x is not None}

  squares_generator = (x**2 for x in range(10))

  unique_names = {user.name for user in users if user is not None}

  eat(jelly_bean for jelly_bean in jelly_beans
      if jelly_bean.color == 'black')
```

```python
No:
  result = [complicated_transform(
                x, some_argument=x+1)
            for x in iterable if predicate(x)]

  result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

  return ((x, y, z)
          for x in xrange(5)
          for y in xrange(5)
          if x != y
          for z in xrange(5)
          if y != z)
```

<a id="s2.10-lambda-functions"></a>
<a id="lambda-functions"></a>
### 2.10 Lambda Functions

Okay for one-liners.

<a id="s2.10.1-definition"></a>
#### 2.10.1 Definition

Lambdas define anonymous functions in an expression, as opposed to a statement.
They are often used to define callbacks or operators for higher-order functions
like `map()` and `filter()`.

<a id="s2.10.2-pros"></a>
#### 2.10.2 Pros

Convenient.

<a id="s2.10.3-cons"></a>
#### 2.10.3 Cons

Harder to read and debug than local functions. The lack of names means stack
traces are more difficult to understand. Expressiveness is limited because the
function may only contain an expression.

<a id="s2.10.4-decision"></a>
#### 2.10.4 Decision

Okay to use them for one-liners. If the code inside the lambda function is
longer than 60-80 chars, it's probably better to define it as a regular [nested
function](#lexical-scoping).

For common operations like multiplication, use the functions from the `operator`
module instead of lambda functions. For example, prefer `operator.mul` to
`lambda x, y: x * y`.

<a id="s2.11-conditional-expressions"></a>
<a id="conditional-expressions"></a>
### 2.11 Conditional Expressions

Okay for one-liners.

<a id="s2.11.1-definition"></a>
#### 2.11.1 Definition

Conditional expressions (sometimes called a “ternary operator”) are mechanisms
that provide a shorter syntax for if statements. For example:
`x = 1 if cond else 2`.

<a id="s2.11.2-pros"></a>
#### 2.11.2 Pros

Shorter and more convenient than an if statement.

<a id="s2.11.3-cons"></a>
#### 2.11.3 Cons

May be harder to read than an if statement. The condition may be difficult to
locate if the expression is long.

<a id="s2.11.4-decision"></a>
#### 2.11.4 Decision

Okay to use for one-liners. In other cases prefer to use a complete if
statement.

<a id="s2.13-properties"></a>
<a id="properties"></a>
### 2.13 Properties

Use properties for accessing or setting data where you would normally have used
simple, lightweight accessor or setter methods.

<a id="s2.13.1-definition"></a>
#### 2.13.1 Definition

A way to wrap method calls for getting and setting an attribute as a standard
attribute access when the computation is lightweight.

<a id="s2.13.2-pros"></a>
#### 2.13.2 Pros

Readability is increased by eliminating explicit get and set method calls for
simple attribute access. Allows calculations to be lazy. Considered the Pythonic
way to maintain the interface of a class. In terms of performance, allowing
properties bypasses needing trivial accessor methods when a direct variable
access is reasonable. This also allows accessor methods to be added in the
future without breaking the interface.

<a id="s2.13.3-cons"></a>
#### 2.13.3 Cons

Must inherit from `object` in Python 2. Can hide side-effects much like operator
overloading. Can be confusing for subclasses.

<a id="s2.13.4-decision"></a>
#### 2.13.4 Decision

Use properties in new code to access or set data where you would normally have
used simple, lightweight accessor or setter methods. Properties should be
created with the `@property` [decorator](#s2.17-function-and-method-decorators).

Inheritance with properties can be non-obvious if the property itself is not
overridden. Thus one must make sure that accessor methods are called indirectly
to ensure methods overridden in subclasses are called by the property (using the
Template Method DP).

```python
Yes: import math

     class Square(object):
         """A square with two properties: a writable area and a read-only perimeter.

         To use:
         >>> sq = Square(3)
         >>> sq.area
         9
         >>> sq.perimeter
         12
         >>> sq.area = 16
         >>> sq.side
         4
         >>> sq.perimeter
         16
         """

         def __init__(self, side):
             self.side = side

         @property
         def area(self):
             """Gets or sets the area of the square."""
             return self._get_area()

         @area.setter
         def area(self, area):
             return self._set_area(area)

         def _get_area(self):
             """Indirect accessor to calculate the 'area' property."""
             return self.side ** 2

         def _set_area(self, area):
             """Indirect setter to set the 'area' property."""
             self.side = math.sqrt(area)

         @property
         def perimeter(self):
             return self.side * 4
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

<a id="s2.15-deprecated-language-features"></a>
<a id="deprecated-language-features"></a>
### 2.15 Deprecated Language Features

Use string methods instead of the `string` module where possible. Use function
call syntax instead of `apply`. Use list comprehensions and `for` loops instead
of `filter` and `map` when the function argument would have been an inlined
lambda anyway. Use `for` loops instead of `reduce`.

<a id="s2.15.1-definition"></a>
#### 2.15.1 Definition

Current versions of Python provide alternative constructs that people find
generally preferable.

<a id="s2.15.2-decision"></a>
#### 2.15.2 Decision

We do not use any Python version which does not support these features, so there
is no reason not to use the new styles.

```python
Yes: words = foo.split(':')

     [x[1] for x in my_list if x[2] == 5]

     map(math.sqrt, data)    # Ok. No inlined lambda expression.

     fn(*args, **kwargs)
```

```python
No:  words = string.split(foo, ':')

     map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

     apply(fn, args, kwargs)
```

<a id="s2.16-lexical-scoping"></a>
<a id="lexical-scoping"></a>
### 2.16 Lexical Scoping

Okay to use.

<a id="s2.16.1-definition"></a>
#### 2.16.1 Definition

A nested Python function can refer to variables defined in enclosing functions,
but can not assign to them. Variable bindings are resolved using lexical
scoping, that is, based on the static program text. Any assignment to a name in
a block will cause Python to treat all references to that name as a local
variable, even if the use precedes the assignment. If a global declaration
occurs, the name is treated as a global variable.

An example of the use of this feature is:

```python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

<a id="s2.16.2-pros"></a>
#### 2.16.2 Pros

Often results in clearer, more elegant code. Especially comforting to
experienced Lisp and Scheme (and Haskell and ML and ...) programmers.

<a id="s2.16.3-cons"></a>
#### 2.16.3 Cons

Can lead to confusing bugs. Such as this example based on
[PEP-0227](http://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/):

```python
i = 4
def foo(x):
    def bar():
        print(i, end='')
    # ...
    # A bunch of code here
    # ...
    for i in x:  # Ah, i *is* local to foo, so this is what bar sees
        print(i, end='')
    bar()
```

So `foo([1, 2, 3])` will print `1 2 3 3`, not `1 2 3
4`.

<a id="s2.16.4-decision"></a>
#### 2.16.4 Decision

Okay to use.

<a id="s2.20-modern-python"></a>
<a id="modern-python"></a>
### 2.20 Modern Python: Python 3 and from \_\_future\_\_ imports

Python 3 is here! While not every project is ready to
use it yet, all code should be written to be 3 compatible (and tested under
3 when possible).

<a id="s2.20.1-definition"></a>
#### 2.20.1 Definition

Python 3 is a significant change in the Python language. While existing code is
often written with 2.7 in mind, there are some simple things to do to make code
more explicit about its intentions and thus better prepared for use under Python
3 without modification.

<a id="s2.20.2-pros"></a>
#### 2.20.2 Pros

Code written with Python 3 in mind is more explicit and easier to get running
under Python 3 once all of the dependencies of your project are ready.

<a id="s2.20.3-cons"></a>
#### 2.20.3 Cons

Some people find the additional boilerplate to be ugly. It's unusual to add
imports to a module that doesn't actually require the features added by the
import.

<a id="s2.20.4-decision"></a>
#### 2.20.4 Decision

##### from \_\_future\_\_ imports

Use of `from __future__ import` statements is encouraged. All new code should
contain the following and existing code should be updated to be compatible when
possible:

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

If you are not already familiar with those, read up on each here: [absolute
imports](https://www.python.org/dev/peps/pep-0328/), [new `/` division
behavior](https://www.python.org/dev/peps/pep-0238/), and [the print
function](https://www.python.org/dev/peps/pep-3105/).

Please don't omit or remove these imports, even if they're not currently used
in the module. It is better to always have the future imports in all files
so that they are not forgotten during later edits when someone starts using
such a feature.

There are other `from __future__` import statements. Use them as you see fit. We
do not include `unicode_literals` in our recommendations as it is not a clear
win due to implicit default codec conversion consequences it introduces in many
places within Python 2.7. Most code is better off with explicit use of `b''` and
`u''` bytes and unicode string literals as necessary.

##### The six, future, or past libraries.

When your project needs to actively support use under both Python 2 and 3, use
of these libraries is encouraged as you see fit. They exist to make your code
cleaner and life easier.



<a id="s3-python-style-rules"></a>
<a id="python-style-rules"></a>
## 3 Python Style Rules

<a id="s3.5-blank-lines"></a>
<a id="blank-lines"></a>
### 3.5 Blank Lines

Two blank lines between top-level definitions, be they function or class
definitions. One blank line between method definitions and between the `class`
line and the first method. No blank line following a `def` line. Use single
blank lines as you judge appropriate within functions or methods.

<a id="s3.8-comments"></a>
<a id="comments"></a>
### 3.8 Comments and Docstrings

Be sure to use the right style for module, function, method docstrings and
inline comments.

<a id="s3.8.2-comments-in-modules"></a>
<a id="comments-in-modules"></a>
#### 3.8.2 Modules

Every file should contain license boilerplate. Choose the appropriate
boilerplate for the license used by the project (for example, Apache 2.0, BSD,
LGPL, GPL)

<a id="s3.8.4-comments-in-classes"></a>
<a id="comments-in-classes"></a>
#### 3.8.4 Classes

Classes should have a docstring below the class definition describing the class.
If your class has public attributes, they should be documented here in an
`Attributes` section and follow the same formatting as a
[function's `Args`](#doc-function-args) section.

```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

<!-- The next section is copied from the C++ style guide. -->
<a id="s3.8.6-punctuation-spelling-and-grammar"></a>
<a id="punctuation-spelling-and-grammar"></a>
#### 3.8.6 Punctuation, Spelling and Grammar

Pay attention to punctuation, spelling, and grammar; it is easier to read
well-written comments than badly written ones.

Comments should be as readable as narrative text, with proper capitalization and
punctuation. In many cases, complete sentences are more readable than sentence
fragments. Shorter comments, such as comments at the end of a line of code, can
sometimes be less formal, but you should be consistent with your style.

Although it can be frustrating to have a code reviewer point out that you are
using a comma when you should be using a semicolon, it is very important that
source code maintain a high level of clarity and readability. Proper
punctuation, spelling, and grammar help with that goal.

<a id="s3.10-strings"></a>
<a id="strings"></a>
### 3.10 Strings

Use the `format` method or the `%` operator for formatting strings, even when
the parameters are all strings. Use your best judgement to decide between `+`
and `%` (or `format`) though.

```python
Yes: x = a + b
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}'.format(first, second)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: {}; score: {}'.format(name, n)
     x = f'name: {name}; score: {n}'  # Python 3.6+
```

```python
No: x = '%s%s' % (a, b)  # use + in this case
    x = '{}{}'.format(a, b)  # use + in this case
    x = first + ', ' + second
    x = 'name: ' + name + '; score: ' + str(n)
```

Avoid using the `+` and `+=` operators to accumulate a string within a loop.
Since strings are immutable, this creates unnecessary temporary objects and
results in quadratic rather than linear running time. Instead, add each
substring to a list and `''.join` the list after the loop terminates (or, write
each substring to a `io.BytesIO` buffer).

```python
Yes: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
```

```python
No: employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
```

Be consistent with your choice of string quote character within a file. Pick `'`
or `"` and stick with it. It is okay to use the other quote character on a
string to avoid the need to `\\` escape within the string. `gpylint` enforces
this.

```python
Yes:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
```

```python
No:
  Python("Why are you hiding your eyes?")
  Gollum('The lint. It burns. It burns us.')
  Gollum("Always the great lint. Watching. Watching.")
```

Prefer `"""` for multi-line strings rather than `'''`. Projects may choose to
use `'''` for all non-docstring multi-line strings if and only if they also use
`'` for regular strings. Docstrings must use `"""` regardless. Note that it is
often cleaner to use implicit line joining since multi-line strings do not flow
with the indentation of the rest of the program:

```python
  Yes:
  print("This is much nicer.\n"
        "Do it this way.\n")
```

```python
  No:
    print("""This is pretty ugly.
Don't do this.
""")
```

<a id="s3.11-files-and-sockets"></a>
<a id="files-and-sockets"></a>
### 3.11 Files and Sockets

Explicitly close files and sockets when done with them.

Leaving files, sockets or other file-like objects open unnecessarily has many
downsides:

-   They may consume limited system resources, such as file descriptors. Code
    that deals with many such objects may exhaust those resources unnecessarily
    if they're not returned to the system promptly after use.
-   Holding files open may prevent other actions such as moving or deleting
    them.
-   Files and sockets that are shared throughout a program may inadvertently be
    read from or written to after logically being closed. If they are actually
    closed, attempts to read or write from them will throw exceptions, making
    the problem known sooner.

Furthermore, while files and sockets are automatically closed when the file
object is destructed, tying the lifetime of the file object to the state of the
file is poor practice:

-   There are no guarantees as to when the runtime will actually run the file's
    destructor. Different Python implementations use different memory management
    techniques, such as delayed Garbage Collection, which may increase the
    object's lifetime arbitrarily and indefinitely.
-   Unexpected references to the file, e.g. in globals or exception tracebacks,
    may keep it around longer than intended.

The preferred way to manage files is using the ["with"
statement](http://docs.python.org/reference/compound_stmts.html#the-with-statement):

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

For file-like objects that do not support the "with" statement, use
`contextlib.closing()`:

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

<a id="s3.12-todo-comments"></a>
<a id="todo-comments"></a>
### 3.12 TODO Comments

Use `TODO` comments for code that is temporary, a short-term solution, or
good-enough but not perfect.

A `TODO` comment begins with the string `TODO` in all caps and a parenthesized
name, e-mail address, or other identifier
of the person or issue with the best context about the problem. This is followed
by an explanation of what there is to do.

The purpose is to have a consistent `TODO` format that can be searched to find
out how to get more details. A `TODO` is not a commitment that the person
referenced will fix the problem. Thus when you create a
`TODO`, it is almost always your name
that is given.

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

If your `TODO` is of the form "At a future date do something" make sure that you
either include a very specific date ("Fix by November 2009") or a very specific
event ("Remove this code when all clients can handle XML responses.").

<a id="s3.13-imports-formatting"></a>
<a id="imports-formatting"></a>
### 3.13 Imports formatting

Imports should be on separate lines.

E.g.:

```python
Yes: import os
     import sys
```

```python
No:  import os, sys
```


Imports are always put at the top of the file, just after any module comments
and docstrings and before module globals and constants. Imports should be
grouped from most generic to least generic:

1.  Python standard library imports. For example:

    ```python
    import sys
    ```

2.  [third-party](https://pypi.org/)
    module or package imports. For example:


    ```python
    import tensorflow as tf
    ```

3.  Code repository
    sub-package imports. For example:


    ```python
    from otherproject.ai import mind
    ```

4.  **Deprecated:** application-specific imports that are part of the same
    top level
    sub-package as this file. For example:


    ```python
    from myproject.backend.hgwells import time_machine
    ```

    You may find older Google Python Style code doing this, but it is no longer
    required.  **New code is encouraged not to bother with this.**  Simply
    treat application-specific sub-package imports the same as other
    sub-package imports.


Within each grouping, imports should be sorted lexicographically, ignoring case,
according to each module's full package path. Code may optionally place a blank
line between import sections.

```python
import collections
import queue
import sys

from absl import app
from absl import flags
import bs4
import cryptography
import tensorflow as tf

from book.genres import scifi
from myproject.backend.hgwells import time_machine
from myproject.backend.state_machine import main_loop
from otherproject.ai import body
from otherproject.ai import mind
from otherproject.ai import soul

# Older style code may have these imports down here instead:
#from myproject.backend.hgwells import time_machine
#from myproject.backend.state_machine import main_loop
```


<a id="s3.14-statements"></a>
<a id="statements"></a>
### 3.14 Statements

Generally only one statement per line.

However, you may put the result of a test on the same line as the test only if
the entire statement fits on one line. In particular, you can never do so with
`try`/`except` since the `try` and `except` can't both fit on the same line, and
you can only do so with an `if` if there is no `else`.

```python
Yes:

  if foo: bar(foo)
```

```python
No:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```

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
