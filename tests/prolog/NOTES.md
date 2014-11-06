________________________________________________________________________

This file is part of Logtalk <http://logtalk.org/>  

Logtalk is free software. You can redistribute it and/or modify it under
the terms of the FSF GNU General Public License 3  (plus some additional
terms per section 7).        Consult the `LICENSE.txt` file for details.
________________________________________________________________________


This directory contains a set of unit tests for Prolog official and de facto
standard features. Most of these unit tests are taken from the official ISO
Prolog standard (updated up to the the ISO/IEC 13211-1:1995/Cor.2:2012(en)
standard). Several tests originate from SICS and are used here with permission.

Writing these tests was made easier by rewriting and, whenever necessary,
updating, the tests found on the Prolog ISO conformance testing framework
written by Péter Szabó and Péter Szeredi, who gracefully allowed me to reuse
their hard work. The framework is described in the following paper:

@incollection{
	year={2006},
	isbn={978-3-540-36635-5},
	booktitle={Logic Programming},
	volume={4079},
	series={Lecture Notes in Computer Science},
	editor={Etalle, Sandro and Truszczyński, Mirosław},
	doi={10.1007/11799573_20},
	title={Improving the ISO Prolog Standard by Analyzing Compliance Test Results},
	url={http://dx.doi.org/10.1007/11799573_20},
	publisher={Springer Berlin Heidelberg},
	author={Szabó, Péter and Szeredi, Péter},
	pages={257-269},
	language={English}
}

To run all the provided tests with e.g. SWI-Prolog, open a terminal and type:

	$ cd $LOGTALKUSER/tests/prolog
	$ logtalk_tester -p swi
	...

By convention, tests for standard built-in predicates encapsulate the main
test goal using the `{}/1` control construct. In most cases, this precaution
is not necessary as the calls would be compiled as-is. An exception is the
read and write predicates that are affected by operator declarations, which
otherwise would be compiled to ensure the locality of operators to conform
to Logtalk semantics.

Some unit tests are currently skipped. These tests are mainly split between
two groups: tests whose result is specified as undefined in the standards
(usually due to the potential of creating cyclic terms if unification without
occurs check is performed) and tests that would require a portable way of
specifying a source file text encoding plus a common extended text encoding
such as UTF-8.

Failure of unit tests doesn't necessarily mean that a backend Prolog compiler
is faulty. Standards are not perfect and there isn't always a consensus on
what should be the correct test results. Common causes of failure are corner
cases in arithmetic operations. Moreover, some Prolog compilers provide a
strict ISO mode that may result in different test results. This strict mode,
when made available, is usually only used if it's the default when starting
Logtalk.

To find if the tests are run in strict ISO mode (`iso_strict`) or in lax ISO
mode (`iso_lax`), check the value of the read-only flag `prolog_conformance`
using the query:

	?- current_logtalk_flag(prolog_conformance, Mode).

This flag is used mainly in testing error conditions. For some of those tests,
when running in lax mode, exception terms may be accepted as valid if they are
reasonable although non-compliant.