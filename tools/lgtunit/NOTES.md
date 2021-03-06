
________________________________________________________________________

This file is part of Logtalk <http://logtalk.org/>  
Copyright 1998-2017 Paulo Moura <pmoura@logtalk.org>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
________________________________________________________________________


Overview
--------

The `lgtunit` tool provides unit testing support for Logtalk. It can be used
for testing both Logtalk and Prolog code.

This tool is inspired by the xUnit frameworks architecture and by the works of
Joachim Schimpf (ECLiPSe library `test_util`) and Jan Wielemaker (SWI-Prolog
`plunit` package).


Main files
----------

The `lgtunit.lgt` source file implements a framework for defining and running
unit tests in Logtalk. The `lgtunit_messages.lgt` source file defines the
default translations for the messages printed when running unit tests. These
messages can be intercepted to customize output, e.g. to make it less verbose,
or for integration with e.g. GUI IDEs and continuous integration servers.

Other files part of this tool provide support for alternative output formats
of test results and are discussed below.


API documentation
-----------------

To consult this tool API documentation, open in a web browser the link:

[docs/directory_index.html#tools/lgtunit/](http://logtalk.org/docs/directory_index.html#tools/lgtunit/)


Loading
-------

This tool can be loaded using the query:

	| ?- logtalk_load(lgtunit(loader)).


Compiling and loading unit tests
--------------------------------

In order to write your own unit tests, define objects extending the `lgtunit`
object:

	:- object(tests,
		extends(lgtunit)).

		...

	:- end_object.

The tests must be term-expanded by the `lgtunit` object by compiling the source
files defining the test objects using the option `hook(lgtunit)`. For example:

	| ?- logtalk_load(my_tests, [hook(lgtunit)]).

As the term-expansion mechanism applies to all the contents of a source file,
the source files defining the test objects shouldn't contain entities other
than the test objects. Additional code necessary for the tests should go to
separate files.

See the `../../tester-sample.lgt` file for an example of a loader file for
compiling and loading the `lgtunit` tool, the source code under testing, the
unit tests, and for automatically run all the tests after loading. See e.g.
the `../../tests` directory for examples of unit tests.


Unit test dialects
------------------

Multiple test _dialects_ are supported by default. Other test dialects can
be easily defined by extending the `lgtunit` object and by term-expanding
the new dialect into one of the default dialects.

Unit tests can be written using any of the following dialects:

	test(Test) :- Goal.

This is the most simple dialect, allowing the specification of tests that
are expected to succeed. The argument of the `test/1` predicate, `Test`, is
an atom that identifies it and must be unique. A more versatile dialect is:

	succeeds(Test) :- Goal.
	deterministic(Test) :- Goal.
	fails(Test) :- Goal.
	throws(Test, Ball) :- Goal.
	throws(Test, Balls) :- Goal.

This is a straightforward dialect. For `succeeds/1` tests, `Goal` is
expected to succeed. For `deterministic/1` tests, `Goal` is expected to
succeed once without leaving a choice-point. For `fails/1` tests, `Goal`
is expected to fail. For `throws/2` tests, `Goal` is expected to throw
the exception term `Ball` or one of the exception terms in the list
`Balls`. The specified exception must subsume the generated exception
for the test to succeed.

An alternative test dialect that can be used with the same expressive
power is:

	test(Test, Outcome) :- Goal.

The possible values of the outcome argument are:

- `true`  
	the test is expected to succeed
- `true(Test)`  
	the test is expected to succeed and satisfy the goal `Test`
- `deterministic`  
	the test is expected to succeed once without leaving a choice-point
- `deterministic(Test)`  
	the test is expected to succeed once without leaving a choice-point and satisfy the goal `Test`
- `fail`  
	the test is expected to fail
- `error(Error)`  
	the test is expected to throw the exception term `error(Error, _)`
- `errors(Errors)`  
	the test is expected to throw an exception term `error(Error, _)` where `Error` is an element of the list `Errors`
- `ball(Ball)`  
	the test is expected to throw the exception term `Ball`
- `balls(Balls)`  
	the test is expected to throw an exception term `Ball` where `Ball` is an element of the list `Balls`

Some tests may require individual setup and/or cleanup goals. in this case,
the following alternative test dialect can be used:

	test(Test, Outcome, Options) :- Goal.

The currently supported options are (non-recognized options are ignored):

- `condition(Goal)`  
	condition for deciding if the test should be run or skipped (default goal is `true`)
- `setup(Goal)`  
	setup goal for the test (default goal is `true`)
- `cleanup(Goal)`  
	cleanup goal for the test (default goal is `true`)
- `note(Term)`  
	annotation to print (between parenthesis by default) after the test result (default is `''`); the annotation term can share variables with the test goal, which can be used to pass additional information about the test result

Also supported is QuickCheck testing where random tests are automatically
generated and run given a predicate mode template with type information for
each argument (see the section below for more details):

	quick_check(Test, Template, Options).
	quick_check(Test, Template).

The valid options are the same as for the `test/3` dialect plus a `n/1` option
to specify the number of random tests that will be generated and run (defaults
to 100 tests).

In all dialects, `Test` is an atom, uniquely identifying a test. An error
message is printed if duplicated identifiers are found. These errors must
be corrected otherwise the test results can be misleading.

For examples of how to write unit tests, check the `tests` folder or the
`testing` example in the `examples` folder in the Logtalk distribution.
Most of the provided examples also include unit tests, some of them with
code coverage.

Parameterized unit tests can be easily defined by using parametric objects.

Note: when using the `(<<)/2` debugging control construct to access and test
an object internal predicates, make sure that the `context_switching_calls`
compiler flag is set to `allow` for those objects.


QuickCheck
----------

QuickCheck was originally developed for Haskell. Implementations for several
other programming languages soon followed. The idea is to express properties
that predicates must comply with and automatically generate tests for those
properties. The `lgtunit` tool supports both `quick_check/2-3` test idioms, as
described above, and `quick_check/1-3` public predicates for interactive use:

	quick_check(Template, Options, Result).
	quick_check(Template, Options).
	quick_check(Template).

The `quick_check/3` predicate returns results in reified form (either `passed`
or `failed(Goal)` with Goal being the random test that failed). The other two
predicates print the test results. The template can be a `::/2`, `<</2`, or
`:/2` qualified callable term. When the template is an unqualified callable
term, it will be used to construct a goal to be called in the context of the
*sender* using the `<</2` debugging control construct.

Properties are expressed using predicates. The QuickCheck test idioms and
predicates take as argument the mode template for a property (defined as a
local predicate) and generate random values for each input argument based
on the type information. The mode template syntax is the same used in the
`info/2` predicate directives. An optional argument, `n/1`, allows the
specification of the number of random tests that will be generated and run.
The user can define new types to use in the property mode templates to use
with its QuickCheck tests by defining clauses for the `arbitrary` library
category multifile predicates.


Skipping unit tests
-------------------

A unit test object can define the `condition/0` predicate (which defaults to
`true`) to test if some necessary condition for running the tests holds. The
tests are skipped if the call to this predicate fails or generates an error.

Individual tests that for some reason should be unconditionally skipped can
have the test clause head prefixed with the `(-)/1` operator. The number of
skipped tests is reported together with the numbers of passed and failed tests.
To skip a test depending on some condition, use the `test/3` dialect and the
`condition/1` option. The conditional compilation directives can also be used
in alternative but note that in this case there will be no report on the number
of skipped tests.


Testing non-deterministic predicates
------------------------------------

For testing non-deterministic predicates (with a finite and manageable number
of solutions), you can wrap the test goal using the standard `findall/3`
predicate to collect all solutions and check against the list of expected
solutions. When the expected solutions are a set, use in alternative the
standard `setof/3` predicate.


Testing input/output predicates
-------------------------------

Extensive support for testing input/output predicates is provided, based on
similar support found on the Prolog conformance testing framework written by
Péter Szabó and Péter Szeredi.

Two sets of predicates are provided, one for testing text input/output and
one for testing binary input/output. In both cases, temporary files (possibly
referenced by a user-defined alias) are used. The predicates allow setting,
checking, and cleaning text/binary input/output. There is also a small set of
helper predicates for dealing with stream handles and stream positions.

For practical examples, check the included tests for Prolog conformance of
standard input/output predicates.


Unit tests with timeout limits
------------------------------

There's no portable way to call a goal with a timeout limit. However, some
backend Prolog compilers provide this functionality:

- ECLiPSe: `timeout/3` and `timeout/7` library predicates
- SICStus Prolog: `time_out/3` library predicate
- SWI-Prolog: `call_with_time_limit/2` library predicate
- YAP: `time_out/3` library predicate


Setup and cleanup goals
-----------------------

A unit test object can define `setup/0` and `cleanup/0`. goals. The `setup/0`
predicate is called, when defined, before running the object unit tests. The
`cleanup/0` predicate is called, when defined, after running all the object
unit tests. The tests are skipped when the setup goal fails or throws an error.

Per test setup and cleanup goals can be defined using the `test/3` dialect and
the `setup/1` and `cleanup/1` options. The test is skipped when the setup goal
fails or throws an error. Note that a broken test cleanup goal doesn't affect
the test but may adversely affect any following tests.


Test annotations
----------------

It's possible to define per unit and per test annotations to be printed after
the test results or when tests are skipped. This is particularly useful when
some units or some unit tests may be run while still being developed.
Annotations can be used to pass additional information to a user reviewing
test results. By intercepting the unit test framework message printing calls
(using the `message_hook/4` hook predicate), test automation scripts and
integrating tools can also access these annotations.

Units can define a global annotation using the predicate `note/1`. To define
per test annotations, use the `test/3` dialect and the `note/1` option.

Annotations are written, by default, between parenthesis after and in the
same line as the test results.


Debugging failed unit tests
---------------------------

In order to debug failed unit tests, you can compile the unit test objects
without using the `hook/1` option and use the `(<<)/2` debugging control
construct to call the individual tests. For example, assuming you have a
`test(test_n)` unit test that is failing, you can type:

	| ?- logtalk_load(my_tests).
	...

	| ?- my_tests << test(test_n).
	...

You may also compile the unit test objects in debug mode and use the Logtalk
debugger. For example:

	| ?- logtalk_load(debugger(loader)).
	...

	| ?- logtalk_load(my_tests, [debug(on)]).
	...

	| ?- debugger::trace.
	...

	| ?- my_tests << test(test_n).
	...


Code coverage
-------------

If you want entity predicate clause coverage information to be collected
and printed, you will need to compile the entities that you're testing
using the flags `debug(on)` and `source_data(on)`. Be aware, however,
that compiling in debug mode results in a performance penalty.

A single unit test object my include tests for one or more entities (objects,
protocols, and categories). The entities being tested by an unit test object
for which code coverage information should be collected must be declared using
the `cover/1` predicate.

In the printed predicate clause coverage information, you may get a total
number of clauses smaller than the covered clauses. This results from the
use of dynamic predicates with clauses asserted at runtime. You may easily
identify dynamic predicates in the results as their clauses often have a
initial count equal to zero.

The list of indexes of the covered predicate clauses can be quite long.
Some backend Prolog compilers provide a flag or a predicate to control
the depth of printed terms that can be useful:

- CxProlog: `write_depth/2` predicate
- ECLiPSe: `print_depth` flag
- SICStus Prolog: `toplevel_print_options` flag
- SWI-Prolog 7.1.10 or earlier: `toplevel_print_options` flag
- SWI-Prolog 7.1.11 or later: `answer_write_options` flag
- XSB: `set_file_write_depth/1` predicate
- YAP: `write_depth/2-3` predicates


Automating running unit tests
-----------------------------

You can use the `scripts/logtalk_tester.sh` Bash shell script for automating
running unit tests. See the `scripts/NOTES.md` file for details.


Utility predicates
------------------

The `lgtunit` tool provides some utility predicates to simplify writing unit
tests:

- `variant/2` - to check when two terms are a variant of each other (e.g. to
check expected test results against the actual results when they contain variables)
- `Float1 =~= Float2` - for approximate float comparison
- `benchmark(Goal, Time)`
- `benchmark(Goal, Repetitions, Time)`

As the `benchmark/2-3` predicates are meta-predicates, turning on the
`optimize` compiler flag is advised to avoid runtime compilation of the
meta-argument, which would add an overhead to the timing results.

Consult the `lgtunit` object documentation (`docs/tools.html`) for further
details on these predicates.


Exporting unit test results in xUnit XML format
-----------------------------------------------

To output test results in the xUnit XML format, simply load the
`xunit_output.lgt` file before running the tests. This file defines
an object, `xunit_output`, that intercepts and rewrites unit test
execution messages, converting them to the xUnit XML format.

To write the test results to a file using the xUnit XML format, simply
load the `xunit_report.lgt` file before running the tests. A file
named `xunit_report.xml` will be created in the same directory as the
object defining the tests.


Writing unit test results in the TAP output format
--------------------------------------------------

To output test results in the TAP (Test Anything Protocol) format, simply
load the `tap_output.lgt` file before running the tests. This file defines
an object, `tap_output`, that intercepts and rewrites unit test execution
messages, converting them to the TAP output format.

To write the test results to a file using the TAP (Test Anything Protocol)
output format, load instead the `tap_report.lgt` file before running the
tests. A file named `tap_report.txt` will be created in the same directory
as the object defining the tests.

When using the `test/3` dialect with the TAP format, a `note/1` option
whose argument is an atom starting with a `TODO` or `todo` word results
in a test report with a TAP TODO directive.


Known issues
------------

Deterministic unit tests are currently not available when using Lean Prolog
or Quintus Prolog as backend compilers do the lack of a required built-in
support that cannot be sensibly defined in Prolog.

Code coverage is only available when testing Logtalk code.


Other notes
-----------

All source files are formatted using tabs (the recommended setting is a
tab width equivalent to 4 spaces).
