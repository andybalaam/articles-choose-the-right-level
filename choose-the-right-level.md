# Testing: choose the right level

When someone complains that their tests are not providing the benefits that
they were promised, or are more trouble than they're worth, some of us may be
inclined to nod wisely and muse that testing is a skill that must be learned.

Testing is a skill, bad tests are barely better than no tests at all, and
sometimes writing tests is painful.

But sometimes, testing is easy and good, and we feel productive and safe in the
knowledge that our code must be correct, because the tests specify our
requirements simply, and unambiguously.

This article will argue that the most important difference between tests being
a pain or a joy is the level at which we have chosen to write them.

We will begin by defining what we mean by a level, then cover how to
choose what level to test at, and the importance of testing at multiple levels.
Finally we will look at some examples of times the author has seen the
practical effects of the choice of level on the experience of testing
software.

## What is a level?

Each test will run the code under test by calling it via an interface, and may
also insert test code via "seams"[Feathers].  Choosing a "level" for your tests
simply means deciding what group of interfaces and seams you will use to invoke
the code under test.

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
by power we mean ability to express meaningful test cases.

### Not too wide

The best level to test your code will not be too wide - your tests should not be
prone to failure because of the behaviour of uninteresting areas of code.
Uninteresting areas could simply be different components, or third-party
libraries or services.

You can tell tests are too wide when:

* there are many possible reasons why a test has failed.
* it is hard to debug failures because lots of different systems need to be
  observed.
* tests run too slowly to give useful feedback.

When tests are too wide you should consider whether there are more direct
interfaces that may be used to exercise the code under test.  An important
advantage of the practice of test-driven development [TDD] is that it tends to
lead us to build more such interfaces into our design.

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
class under test.  This causes problems when the code under test changes,
and tests fail because the mocks no longer express the true behaviour of
related classes.  More description of how to avoid complex mocks may be found
in [Balaam].

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
HTTP requests in response to method calls, and provides an API to filter the
list of metrics emitted.

Testing the filtering logic we provided to the library was tricky because it
was highly interlinked with the behaviour of the library, and there was no
mechanism available to ask which metrics would be emitted, or provide a
mock HTTP client to track the outcome without actually making HTTP requests.

We were left with a choice between unit testing the filtering logic, accepting
that the assertions we were making were making sweeping assumptions about the
library behaviour (too narrow), or testing the logic in an environment that
actually allowed and tracked the real HTTP requests (too wide).

We eventually added assertions to a test within our full system test
environment that actually included the metrics repository as well as the
messaging component, checking that the repository had received only the correct
subset of metrics.

This seemed most unsatisfactory, and emphasises the need for library writers to
consider the question not only of how to test their own code, but also how
users of the library will test theirs [?]: ideally we would be able to provide
the filtering logic to the metrics library and ask it to list which metrics
will be emitted.

[?] TODO: does anyone know a good reference for this?  I feel like I have
    read this idea somewhere.

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
design, since our team of experienced TDD-ers were uncomfortable writing
too much code without a direct test, so they tended to break complex code into
multiple tools to allow coherent testing.

System level testing made it easy to test features that could easily be
overlooked such as command-line argument processing and formatting of output.

We found this level to be ideal for testing this kind of tool, and rarely
felt concerned by the lack of a pure unit test layer.  Sometimes when writing
Python code we wanted to write unit tests, but this was often addressed by
breaking complex code into multiple executables.  When writing tools in Bash
and other less naturally testable environments, we felt liberated by the fact
that our test setup already made writing tests easy.

Most of the tools involved consumed standard input and emitted results to
standard output, but where they interfaced with external systems such as the
file system or the network, our approach made it hard to test.  We limited such
tools to be as simple as possible and did not include them in our test
coverage, which was not ideal, but rarely caused problems in practice.

Usually, testing only at the system level will cause tests to run too
slowly to be useful, but in this case our suite of hundreds of tools completed
its test run in under 10 seconds, which was good enough for our development
process.

### Game logic

In a rabbit-focussed Android puzzle game [RabbitEscape] we needed to test large
numbers of scenarios in the game model, including interactions between
different rabbit behaviours and user actions (e.g. if I give a rabbit the
"climbing" ability while it's building a bridge, will it stop building?).

Since the game model operates in discrete time steps, using a coarse grid of
spacial positions, we chose to represent game states in ASCII art-style text
grids.  This allows us to express tests of behaviour as sequences of text
"pictures" of the successive game states where e.g. a rabbit is represented as
an 'r' character, and a floor block is represented by '#'.

Writing tests of the game-model component at this level, encoded in this way,
has been remarkably successful.  It is easy to read and understand old tests,
and turning a bug report into a failing unit test is a satisfying and clear
process.

The code base contains plenty of "normal" unit tests, but where we need to test
the in-game behaviour we always turn to this method for its clarity and ease of
writing tests.  There is some run-time overhead in parsing and rendering text
representations of the game state, but this has not caused us a performance
problem so far (hundreds of tests run in about 5 seconds).

One concern in this approach is that tests might become too wide - when testing
a single behaviour or bug we instantiate a whole game world containing all
the game logic, as well as the text parsing and rendering, meaning our tests
could fail for reasons irrelevant to the logic we want to test.  In practice,
although tests do sometimes start failing unexpectedly, more often than not
this is because of an interaction we did not anticipate, that we are glad to
find out about, so the extra coverage of the tests generally works to our
advantage.

Compared with most test frameworks we have seen, this one is a joy to work
with: we are very happy with way it works - almost no test feels pointless, and
newcomers can read and understand the tests very easily.

### Object model

In a JavaScript object model representing domain objects for an interactive
charting application, we needed to implement logic to synchronise changes
between client and server.

We chose to test purely at the unit level, where we took "unit" to mean a piece
of functionality, rather than an object or module.  Most test modules contained
multiple tests exercising the same 1-3 objects from the code under test,
exercising different usage of the same code.  Each test expressed a single
idea, and each test module served as a specification for that behaviour.

Writing the tests and code in this way was a pleasure, and the functionality
was well-covered and easy to understand, partly because of this approach.  One
advantage of viewing units in this way is that it pushes towards better design:
if a test module began to involve more than about three objects, we took this
as a prompt to revisit the design and see whether the behaviour could be
implemented in a more coherent, simpler structure.

One result of choosing this level was that the interface between client and
server was not covered by these tests.  Separately, we built tests that
instantiated a multi-language environment to test the client and server
interactions, effectively running a JavaScript interpreter within the server
platform.  These tests were unreliable and failures were hard to understand
and debug.

In retrospect, our full system test environment (including a full web browser
and a server environment) might have been a more pragmatic way to ensure the
client-server communication worked, instead of trying to cover it explicitly
using headless tests.  Certainly, separating the pure logic into narrow tests
that touched no file system or network resources proved highly effective for
giving us confidence in the logic, but did not cover enough to provide full
confidence in the system as a whole.

### Web service

Working in a large, unfamiliar legacy Java code base for a set of web APIs, we
found an attempt had been made to test at a component level.

Most of the tests were JUnit methods that relied on a complex stack of mock
objects that, when correctly instantiated, replaced the code that made external
connections (e.g. TCP sockets, files) in the production code.

Because the system's observable behaviour consisted mostly of network traffic,
we knew we would need plenty of system-level tests to convince ourselves that
it was behaving correctly.  However, the existing mock-based tests were a
perfect of example of testing at the wrong level: they were not wide enough to
cover the real-world behaviour (e.g. what happens when we encounter a
poorly-configured DNS server), but they had all the down-sides of system-level
tests: they were unreliable, timing-dependent and slow, and depended on the
system being in the correct starting state to run correctly.

At the same time, creating new tests was slow and error-prone due to the large
number of mock classes needed, and tests often broke when the assumptions
encoded in the mock structure became invalid due to changes in the code under
test.  Effectively we were testing the mocks more than the interesting code.

Furthermore, many of the components in this system were tightly integrated with
other components, meaning it was difficult to be sure they were working
correctly without a true system test that ran lots of parts simultaneously.

Our team replaced the heavily-mocked component tests one by one with true
system-level tests that instantiated different subsets of the full production
set-up inside repeatable Docker-based environments, and exercised the system
through its real network interfaces.  Meanwhile, we gradually increased the
coverage of true unit-level testing by writing unit tests whenever we changed
the code.  Within months, our test runs were more reliable, tests were
effective at finding real problems, and the failure rate of the production
software reduced.

By changing the level at which we were testing, from the complex Java
interfaces of the external components to the simple and relatively
slow-changing external HTTP interfaces, we simplified the job of testing,
making it much clearer what the expected input and output were.  Where fake
or mock services were needed, they were simple independent code bases, or
often could be implemented to provide hard-coded HTTP responses, using the
Python http.server module.

The Docker-based tests were slow - even slower than the old mock-based tests -
and they were not 100% reliable, but the significant improvement in
reliability, and the much better coverage of real-world scenarios, was well
worth the extra time.  The far better comprehensiblity of the tests was perhaps
the key advantage longer term, as we worked to understand this complex legacy
system.

### Tree merge

In a large C++ UI application, functionality existed to merge two existing
models based on a large set of rules for how to combine potentially clashing
hierarchical trees.  Building up models in code was complex and verbose, and
deciding whether the produced model was actually correct was difficult.

We took the decision to describe object models in a custom domain-specific
language (DSL).  This allowed us to write tests that clearly described the
input and output conditions without the noise of boilerplate code interfering.
The DSL took only one line to describe each object in the tree, meaning most
of our tests became only one or two screens of code, and the expected behaviour
was clear.

Using a custom DSL has many potential disadvantages, such as the lack of a
debugger, but we had great success using one to describe object models, perhaps
because instantiating objects is a simple enough process that it does not need
to be stepped through line by line.  We could have taken the route of writing
simple functions to create objects, and stayed inside main language, but the
key advantage of the DSL in this case was that test failures produced a clear
diff (in the notation of the DSL) showing what object model was expected and
what was actually seen.  This made interpreting test failures so much simpler
that we could write in a test-driven development style, writing a test and
using the failure to drive changes in the code under test.  This way of working
was an order of magnitude more productive than debugging individual assertion
failures which gave no overall picture of the difference between expected and
actual behaviour.

By changing the level of testing to a wider level (complete input and expected
output, instead of hand-coded variations on an input model and expected
features of the resulting output) we greatly simplified our tests and made them
much more useful.  The use of a DSL was helpful, but less important than the
shift in testing level to one that naturally suited the problem.

## Conclusion

When deciding how to test your code, it is important to consider what level
makes sense for the project.

You should try to choose a level where you can:

1. express your requirements simply.
2. run the important parts of the code (not mock them out).
3. easily interpret test failures.

You can tell when tests are at the right level when:

* No test feels pointless - each test verifies some real part of the spec.
* There are no gaps where the really interesting stuff happens but can't
  be tested.
* It is easy to write the next test.

You can tell the level is wrong when:

* Tests consist of more setup than actual test code.
* Getting your mock working is harder than writing the real code.
* Tests are unreliable.
* The real interesting behaviour is not tested.
* Adding another similar test is hard.
* It takes too long to run your tests.

Often you will need two levels to cover specific code units and whole-system
behaviour.  Large code bases may.need more than two levels - where this is
needed, try to find a level that lets you view a component as having clear
inputs and outputs.

[Balaam] Balaam, A.J. (2015) "Mocks are Bad, Layers are Bad" In F. Buontempo,
editor, Overload 127, pages 8-11.

[Feathers] Feathers, M. (2004) "Working Effectively with Legacy Code", Prentice
Hall

[RabbitEscape] Rabbit Escape, http://artificialworlds.net/rabbit-escape

[TDD] Beck, K.(2002) "Test-Driven Development by Example", Addison Wesley
