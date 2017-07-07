# Test at the right level

Testing is hard, we all know that.

But sometimes testing is easy, especially when we are adding new tests.

In fact, "breaking in" to an area is always the hardest part.

Why?  Because the hard part is choosing the level at which to test.

## Choosing a level - not too wide

too many reasons for failure

too hard to debug failures

too slow

## Choosing a level - not too narrow

doesn't test the actual interesting part

Bugs in production greeted with a chorus of "but the tests passed!"

## Choosing a level - too much setup

Mocks upon mocks upon mocks.

## Choose multiple levels

Test basic logic and correctness at narrow level

Test things that could really go wrong with one or two tests at a wider level

## Examples

### Command line tools

Tested at system level only:

* definitely catches real bugs e.g. with arg parsing, stdin/out
* pushes towards smaller tools, which we want
* reasonably fast

### Metrics filtering

* Can unit test my filtering logic, but it is trivial
* Real question is: does the framework work the way I think it does?
* Real functionality only testable at system level

This is why we like libraries instead of frameworks

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
