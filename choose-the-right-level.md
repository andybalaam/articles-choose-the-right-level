# Testing: choose the right level

When someone complains that their tests aren't providing the benefits that they
were promised, or are more trouble than they're worth, some of us may be
inclined to nod wisely and muse that testing is a skill that must be learned.
We may even hint at the long years we spent sitting at the foot of a mystic
testing master to acquire a few of the deep secrets of the art.

Testing is indeed hard, and it isn't always the magic bullet it is sometimes
portrayed as.

But sometimes, testing is easy and good, and we feel productive and safe in the
knowledge that our code must be correct, because the tests specify our
requirements simply, and unambiguously.

I argue that the most important difference between tests being a pain or a joy
is the level at which we have chosen to write them.

This article will begin by defining what we mean by a level, then cover how
to choose what level to test at, and importance of testing at multiple levels.
Finally we will look at some examples of times the author has seen the
practical effects of the choice of level on the experience of testing software..

## What is a level?

Each test will run the code under test by calling it via an interface, and may
also insert test code via "seams"[1].  Choosing a "level" for your tests simply
means decided what group of interfaces and seams you will use to invoke the
code under test.

[1] TODO test seams Fowler?

Examples of test levels include:

* calling individual methods and checking return values.
* constructing a few independent classes, calling methods on them, and checking
  object state or return values.
* instantiating a large group of classes backed by mocks and checking the
  inputs and outputs via high-level method calls.
* expressing tests as expected output when certain input is provided to the
  executable under test.
* exercising full running systems via external interfaces e.g. HTTP.

Note that several of the examples above concern object-oriented code, but the
idea of choosing a level applies to other coding styles too.

## Choosing a level

Often the best way to choose a test level is to try and write some tests at
different levels and discover which test combines simplicity and power, where
by "power" here we mean ability to express meaningful test cases.

### Not too wide

The best level to test you code will not be too wide - your tests should not be
prone to failure because of the behaviour of areas of code that you are not
interested in.  Areas you are not interested in could simply be different
components, or third-party libraries or services.

You can tell tests are too wide when:

* there are many possible reasons why a test has failed.
* it is hard to debug failures because lots of different systems need to be
  observed.
* tests run too slowly to give useful feedback.

When tests are too wide you should consider whether there are more direct
interfaces that may be used to exercise the code under test.  An important
advantage of the practice of test-driven development [2] is that it tends to
lead us to build more such interfaces into our design.

[2] TODO: TDD

### Not too narrow

Tests should not be too narrow: the interesting and difficult parts of the code
must not be left out at test time.  Where we are implementing some pure logic
it is often easy to write tests that cover that logic, but where we are making
use of an external component, and the most common problems will be caused by
misunderstandings of that component's behaviour, it is more difficult to write
a wide enough test.

You can tell tests are too narrow when:

* production bugs are greeted with a chorus of "but the tests passed!"
* code that mocks external components contains encoded assumptions about how
  those components work.

### Not too much setup

Our tests often consist of "given" (set-up), "when" (doing something) and
"then" (assertions) phases.  When the "given" part is long, error-prone and
complex, we know we have chosen a level that requires too much setup.

A particularly troublesome variant of this situation is when we must instantiate
large numbers of interconnected mock classes just to create an instance of a
class under test.  This causes problems when the the code under test changes,
and tests fail because the mocks no longer express the true behaviour of
related classes.  More description of how to avoid complex mocks may be found
in [3]

[3] TODO Mocks are bad, layers are bad

You can tell when tests have too much setup when:

* each test has a long, complex "given" phase at the beginning.
* it becomes necessary to write test fixture classes just to hold on to all
  the state needed to start a test.
* the tests are dependent on the details of multiple interconnected mock
  classes.

## Choose multiple levels

Normally, a single level of testing will not provide sufficient coverage of
the interesting parts of our code.  It is almost always necessary to write
tests at the full-system level in order to have confidence that the system
works, but the full-system level is usually too wide to allow efficient testing
of all the details of behaviour.

Writing a few simple, real-world tests at the system level (ensuring build and
integration issues are discovered early), and large numbers of detailed
correctness tests at the code level (written as the code is written) can often
be sufficient.  In more complex systems it often makes sense to write tests of
larger chunks of logic at an intermediate level, and here is where choosing the
right level can be tricky.  The best level will allow writing "acceptance"
tests where we can express genuine customer requirements as individual tests,
and run those tests in a reasonable time, without excluding the code that is
most likely to contain bugs.

## Examples

The following sections describe examples of real-world projects that involved
choosing a level at which to test, and an assessment of the success of that
choice.

### Reporting metrics

In a high-availability messaging product written in Java that emitted a large
set of metrics for real-time monitoring, the team found that the metrics
repository was becoming overwhelmed, so we needed to reduce the number of
individual metrics being emitted.

Metrics reporting was provided by a third-party library which directly makes
HTTP requests in response to method calls, and provides an API to filter which
metrics are emitted.

Testing the filtering logic we provided to the library was tricky because it
was highly interlinked with the behaviour of the library, and there was no
mechanism available to ask the which metrics would be emitted, or provide a
mock HTTP client to track the outcome without actually making HTTP requests.

We were left with a choice between unit testing the filtering logic, accepting
that the assertions we were making were making sweeping assumptions about the
library behaviour, or testing the logic in an environment that actually
allowed and tracked the real HTTP requests.

We eventually added an assertion to a test within our full system test
environment that actually included the metrics repository as well as the
messaging component, checking that the repository had received only the correct
subset of metrics.

This seemed most unsatisfactory, and emphasises the need for library writers to
consider the question not only of how to test their own code, but also how
users of the library will test theirs [3]: ideally we would be able to provide
the filtering logic to the metrics library and ask it to list which metrics
will be emitted.

[3] TODO reference on library writers considering how to test code using it

### Command line tools

In a team maintaining a suite of auxiliary utility tools used on the command
line, we settled on a strategy of writing tests that invoked the actual
executable commands with pre-specified input and expected output.  The tools
were written in a variety of technologies, and the tests were written in
Python, executing the commands using the subprocess module.

One of the design goals of these tools was to ensure each executable had a
simple, well-defined job so that tools could be combined in flexible pipelines
to handle unexpected scenarios.

The choice of using only system-level testing encouraged us to follow this
design goal, since this team of experienced TDD-ers were uncomfortable writing
too much code without a direct test, so they tended to break complex code into
multiple parts to allow coherent testing.

Additionally, system level testing made it easy to test features that could
easily be overlooked such as command-line argument processing and formatting
of output.

We found this level to be ideal for testing this kind of tool, and rarely
felt concerned by the lack of a pure unit test layer.  Sometimes when writing
Python code we wanted to write unit tests, but this was often addressed by
breaking complex code into multiple executables.  When writing tools in Bash
and other less naturally testable environments, we felt liberated by the fact
that our test setup already made writing tests easy.

Downsides: testing scripts that really do things

Potential downside: speed, but was ok.

### Game logic

Simple rabbit-based game

Tests of rabbit behaviour are almost all done by constructing a whole (small)
level using ASCII art.

* Easy to understand tests
* Very clear that the behaviour really works
* Fast enough (but clearly not as fast as it could be)

This is a joy to work with.  Very happy with my decision.  Almost no test
feels pointless, and others can read tests very easily.

### Object model

Unit tests, where unit is defined as a unit of functionality, not a file.

Each behaviour involves 1-3 interacting classes.  Tests check that something
really works, not just that a method can be called.

Pushes towards better design: if a test will involve >3 classes, maybe the
design needs revisiting.

### Web service

Legacy codebase - lots of discovery needed to figure out how it works.  These
need to be system-level, or at least "wide" tests, but it still matters which
seam or level we choose.

Integration tests: some in Java with heavy mocking, some in Docker with
real HTTP.

* Java+mocks test are unreliable - all the downsides of system tests (rely on
  timing, slow, depend on global state)
* Java+mocks tests are really hard to understand - 95% setup of various hairy
  mocks that must work perfectly for test to pass.  We are mostly testing
  the mocks.
* Docker-based tests exercise service through HTTP interface, which is
  well-defined and simple.
* Simulator/mock services are separate from tests and relatively easy to
  understand.  Some of them are just python3 -m http.server
* Docker-based tests are slow - even slower than Java+mocks.
* Docker-based tests are more reliable.  No use of global state, and we can
  wait for services to be ready via that clear interface of HTTP.

## Conclusion

Choose a level where you can:
a) express your requirements simply
b) make the code actually work without thousands of mocks

Pick the right level.  You can tell it's right when:

* No test feels pointless - each test verifies some real part of the spec.
* There are no gaps where the really interesting stuff happens but can't
  be tested
* It is easy to write the next test

You can tell it's wrong when:

* Tests consist of more setup than actual test code
* Getting your mock working is harder than writing the real code
* Tests are unreliable
* The real interesting behaviour is not tested
* Adding another similar test is hard
* It takes ages to run your tests

Often you will need 2 levels.

Avoid frameworks.
