# A Beginner's Guide to Property-Based Testing 

[![CI](https://github.com/meeshkan/introduction-to-property-based-testing/workflows/CI/badge.svg)](https://github.com/meeshkan/introduction-to-property-based-testing/actions?query=branch%3Amaster)
[![Chat on Gitter](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/meeshkan/community)

This repository contains the [article draft](ARTICLE.md) introducing property-based testing and working test code in python.

We've created a [GitHub repository](https://github.com/meeshkan/beginners-guide-to-property-based-testing) to go with this guide. All of the featured code examples exist there as unit tests and include instructions for how to execute them.

The full guide associated with this repository 

[Chat with us on Gitter](https://gitter.im/meeshkan/community) if you run into problems or have any questions!

## Table of Contents
- [Running the tests](#running-the-tests)
- [Available libraries](#available-libraries)
- [More resources](#more-resources)
- [Contributing](#contributing)
- [Tell us what you think](#tell-us-what-you-think)

## Running the tests

⚠️ **Prerequisites**:
- [Python 3+](https://www.python.org/downloads/)

Clone this repository and move into the directory:
```bash
git clone https://github.com/meeshkan/beginners-guide-to-property-based-testing.git
cd beginners-guide-to-property-based-testing
```

Then, run `make`:

```bash
make
```

> This command executes scripts from the [Makefile](./Makefile). These scripts set up a [`venv` directory](https://docs.python.org/3/library/venv.html), install the dependencies and run the tests. It will also automatically format the code using [black](https://black.readthedocs.io/en/stable/) and [isort](https://timothycrosley.github.io/isort/).

After you've run `make` once, you can also execute the tests using the following steps.

Launch your virtual environment:

```bash
. venv/bin/activate
```

> Whenever you're done, you can terminate the virtual environment by running: `deactivate`  

Then, run [pytest](https://pypi.org/project/pytest/):

```bash
pytest
```

You can also run individual tests with:

```bash
pytest file_name.py
```

## Available libraries

- [Hypothesis](https://hypothesis.readthedocs.io/en/latest/): Python (used in our guide)
- [fast-check](https://github.com/dubzzz/fast-check): TypeScript
- [FsCheck](https://fscheck.github.io/FsCheck/): .NET
- [jqwik](https://jqwik.net/): Java
- [PropCheck](https://github.com/alfert/propcheck): Elixir
- [PropEr](https://proper-testing.github.io/): Erlang
- [RapidCheck](https://github.com/emil-e/rapidcheck): C++
- [QuickCheck](https://hackage.haskell.org/package/QuickCheck): Haskell
- [QuickCheck ported to Rust](https://docs.rs/quickcheck/0.9.2/quickcheck/): Rust

## More resources

- https://hackage.haskell.org/package/QuickCheck
  - Early (1999) Haskell library.
  - https://jqwik.net/property-based-testing.html mentions it as "Quickcheck is the original tool for writing property tests."
  - Also https://en.wikipedia.org/wiki/QuickCheck: "QuickCheck is a software library, specifically a combinator library, originally written in the programming language Haskell, designed to assist in software testing by generating test cases for test suites. It is compatible with the compiler, Glasgow Haskell Compiler (GHC) and the interpreter, Haskell User's Gofer System (Hugs). It is free and open-source software released under a BSD-style license. In QuickCheck, assertions are written about logical properties that a function should fulfill. Then QuickCheck attempts to generate a test case that falsifies such assertions. Once such a test case is found, QuickCheck tries to reduce it to a minimal failing subset by removing or simplifying input data that are unneeded to make the test fail. The project began in 1999. Besides being used to test regular programs, QuickCheck is also useful for building up a functional specification, for documenting what functions should be doing, and for testing compiler implementations."
- https://hypothesis.works/
  - "This sort of testing is often called 'property-based testing', and the most widely known implementation of the concept is the Haskell library QuickCheck, but Hypothesis differs significantly from QuickCheck and is designed to fit idiomatically and easily into existing styles of testing that you are used to, with absolutely no familiarity with Haskell or functional programming needed."
  - https://hypothesis.works/articles/what-is-property-based-testing/
- https://github.com/ksaaskil/introduction-to-property-based-testing
- https://medium.com/criteo-labs/introduction-to-property-based-testing-f5236229d237
- https://jqwik.net/property-based-testing.html
- https://blog.ssanj.net/posts/2016-06-26-property-based-testing-patterns.html

## Contributing

Notice a bug? Interested in adding a new section to our guide? Have any other property-based testing resources you think we should know? The best way to get involved is to [open an issue](https://github.com/meeshkan/beginners-guide-to-property-based-testing/issues).

Please note that this project is governed by the [Meeshkan Community Code of Conduct](https://github.com/meeshkan/code-of-conduct). By participating, you agree to abide by its terms.

## Tell us what you think

At [Meeshkan](https://meeshkan.com/), we're working to improve how people test their products. So no matter if you loved or loathed our guide, we want to hear from you. 

Here are some ways you can get in touch:
- [Open an issue](https://github.com/meeshkan/beginners-guide-to-property-based-testing/issues)
- [Tweet at us](https://twitter.com/meeshkanml)
- [Reach out on Gitter](https://gitter.im/Meeshkan/community)

Some lingering questions we have:
- Why weren't you using property-based testing before?
- After reading through this guide, would you be willing to try? 
- Would you be interested in seeing another article expanding on the topic?