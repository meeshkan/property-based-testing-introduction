[![CI](https://github.com/meeshkan/introduction-to-property-based-testing/workflows/CI/badge.svg)](https://github.com/meeshkan/introduction-to-property-based-testing/actions?query=branch%3Amaster)

# Introduction to property-based testing
This repository contains this README article introducing property-based testing with code samples in python.

## Example based testing
Normally software testing is done through **example based testing**. A human writes one or several sample inputs to the function or system under test, runs the function or system, and then asserts on the result of that.

Let's start with a toy example - a python [bubble sort](https://en.wikipedia.org/wiki/Bubble_sort) function to sort a list of numbers (note: this is a toy example, use [list.sort()](https://docs.python.org/3/library/stdtypes.html#list.sort) in production):

```python
def bubble_sort(nums):
    result = nums.copy()
    swapped = True
    while swapped:
        swapped = False
        for i in range(len(result) - 1):
            if result[i] > result[i + 1]:
                result[i], result[i + 1] = result[i + 1], result[i]
                swapped = True
    return result

def test_bubble_sort_example():
    # Test using two manually worked out examples:
    assert [1, 2, 3, 4, 5] == bubble_sort([5, 3, 1, 4, 2])
    assert [1, 1, 3, 3, 5] == bubble_sort([1, 3, 1, 3, 5])
```

## Property based testing
While great and simple, testing examples does just that: test examples that we have come up with! What if we want to test hundreds of test cases, possibly ones we could never dream of coming up with ourselves?

In other words: How could we have saved the student records in the below example?

![Bobby tables comic about SQL injections](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)

**Property based testing** is a different approach here to help with that. You yourself don't generate the exact input - that is done by by a computer automatically. What you as a developer do is:

- You specify what input to generate.
- You assert on guarantees (here after called **properties**) which are true regardless of exact input.

Let's see an example using the [Hypothesis](https://hypothesis.readthedocs.io/en/latest/) test library:

```python
import hypothesis.strategies as some
from hypothesis import given

# Guide the framework in what input we need:
@given(input_list=some.lists(some.integers()))
def test_bubble_sort_properties(input_list):
    input_list_copy = input_list.copy()
    sorted_list = bubble_sort(input_list)

    # Regardless of input, sorting should never change the original list:
    assert input_list_copy == input_list

    # Regardless of input, sorting should never change the size:
    assert len(sorted_list) == len(input_list)

    # Regardless of input, sorting should never change the set of distinct elements:
    assert set(sorted_list) == set(input_list)

    # Regardless of input, each element in the sorted list should be
    # lower or equal to the value that comes after it:
    for i in range(len(sorted_list) - 1):
        assert sorted_list[i] <= sorted_list[i + 1]
```

Here we specify that we want lists of integers as input (using the [@given](https://hypothesis.readthedocs.io/en/latest/details.html#hypothesis.given) function decorator) and **asserts on properties that are true regardless of the exact input**.

If we peek on what input is generated by adding a `print(input_list)` statement, we can see 100 different generated input values (the number of runs and specifics can be configured, more on that later on):

```
[]
[92]
[66, 24, -25219, 94, -28953, 31131]
[-16316, -367479896]
[-7336253322929551029, -7336253322929551029, 27974, -24308, -64]
...
```

While different, a property based test shares a lot with how an example based test is written.

| Example based                          | Property based                              |
| -------------------------------------- | ------------------------------------------- |
| 1. Set up some data                    | 1. For all data matching some specification |
| 2. Perform some operations on the data | 2. Perform some operations on the data      |
| 3. Assert something about the result   | 3. Assert something about the result        |


## Property: Unexpected exceptions should never be thrown
One thing we got tested "for free" in the above `test_bubble_sort_properties` function, was that the code did not throw any exception. This property, that the code does not throw any exception (or more generally, only expected and documented exceptions), can be a convenient one to test, especially if the code has a lot of internal assertions.

![Comic about computer complaining about segfaults](https://imgs.xkcd.com/comics/compiler_complaint.png)

Let's test that the property that the [json.loads](https://docs.python.org/3/library/json.html#json.loads) function in the python standard library never throws any exception other than `json.JSONDecodeError` regardless of input:

```python
@given(some.text())
def test_json_loads(input_string):
    try:
        json.loads(input_string)
    except json.JSONDecodeError:
        return
```

Running the test passes, so what we believe held up under test!

## Property: Symmetry, such as decoding an encoding value always brings back original
Symmetry of certain operations, such the property that decoding an encoded value always brings back the original value, can sometimes be used.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/68/Simetria-bilateria.svg/2560px-Simetria-bilateria.svg.png" alt="drawing" width="666" height="205"/>

Let's apply it to [base32-crockford](https://github.com/jbittel/base32-crockford), a python library for the [Base 32](https://www.crockford.com/base32.html) 

```python
@given(some.integers(min_value=0))
def test_base32_crockford(input_int):
      assert base32_crockford.decode(base32_crockford.encode(input_int)) == input_int
```

Since this decoding scheme only works for non-negative integers, we specify to the **generator** of input data to only generate integers with a minium value of zero: `some.integers(min_value=0)`.

## Comparing with a correct value obtained another way
Sometimes we can get the desired solution through a way that is not acceptable to use in production code: That might be due to execution time being to slow, memory consumption too high or requiring special dependencies that are not acceptable to install in production.

![From the Matrix movie: What if I told you.. You are absolutely correct](https://memes.ucoz.com/_nw/41/66267655.jpg)

For an example, consider counting the number of bits in an (arbitrary sized) integer, where we have an optimized solution from the [pygmp2](https://gmpy2.readthedocs.io/en/latest/) library, which we compare with a slow solution that converts the integer to a binary string and counting the occurences of the string "1" inside it, using the [bin](https://docs.python.org/3/library/functions.html#bin) function in the standard python library:

```python
def count_bits_slow(input_int):
    return bin(input_int).count("1")

@given(some.integers(min_value=0))
@settings(max_examples=500)
def test_gmpy2_popcount(input_int):
    assert count_bits_slow(input_int) == gmpy2.popcount(input_int)
```

For illustrative purposes we have here specified a [@settings(max_examples=500)](https://hypothesis.readthedocs.io/en/latest/settings.html) decorator to tweak the default number of input values to generate.

# Finding an issue in the wild
We haven't had a failing test yet - let's go hunting!

![Comic about debugging, an actual bug crawling out of a computer](https://img.devrant.com/devrant/rant/r_1495963_9LswA.jpg)

The [json5](https://pypi.org/project/json5/) library for [JSON5](https://json5.org/) serialization might be a good fit (besides being a young project and therefore more likely to contain bugs):

- One **property** of JSON5 is that it is a superset of JSON.
- Another **property** (which is true of most serialization formats) is that deserializing a serialized string should give us back the original object.

Let's use those properties in a test:

```python
import json
from string import printable

import hypothesis.strategies as some
import json5
from hypothesis import example, given, settings

# Construct a generator of arbitrary objects to test serialization on:
some_object = some.recursive(
    some.none() | some.booleans() | some.floats(allow_nan=False) | some.text(printable),
    lambda children: some.lists(children, min_size=1)
    | some.dictionaries(some.text(printable), children, min_size=1),
)


@given(some_object)
def test_json5_loads(input_object):
    dumped_json_string = json.dumps(input_object)
    dumped_json5_string = json5.dumps(input_object)

    parsed_object_from_json = json5.loads(dumped_json_string)
    parsed_object_from_json5 = json5.loads(dumped_json5_string)

    assert parsed_object_from_json == input_object
    assert parsed_object_from_json5 == input_object
```

After creating a `some_object` generator of arbitrary objects (see [the Hypothesis documentation](https://hypothesis.readthedocs.io/en/latest/data.html#recursive-data) for details) we verify aspects of the previosly mentiond properties, by serialising using both `json` and `json5`, then deserialising those two objects back using the `json5` library and asserting that the original object was obtained.

Lo and behold - at the `json5.dumps(input_object)` statement we get an exception inside the internals of the `json5` library:

```python
    def _is_ident(k):
        k = str(k)
>       if not _is_id_start(k[0]) and k[0] not in (u'$', u'_'):
E       IndexError: string index out of range
```

![Our testers found more bugs than our customers did](https://qph.fs.quoracdn.net/main-qimg-c1eb5bad58ac4d222c17196e0a8f2288)

Besides showing the stack trace as usual, we also get an informative message showing the failed **hypothesis**, the generatted data causing our test to fail:

```
------------- Hypothesis -------------
Falsifying example: test_json5_loads(
    input_object={'': None},
)
```

Using the `{'': None}` input data causing the issue it was easy to promptly [report](https://github.com/dpranke/pyjson5/issues/37) and [fix](https://github.com/dpranke/pyjson5/pull/38) the bug, which has since been released in version 0.9.4 of the library.

But what about the future - how can we be sure that the problem never resurfaces? While we saw that we currently generated input contained the troublesome input, we want to ensure that this input is always used, even in the face of someone tweaking the `some_object` generator or updating the version of the Hypothesis library used:

```diff
--- test_json5_decode_orig.py	2020-03-27 09:48:24.000000000 +0100
+++ test_json5_decode.py	2020-03-27 09:48:32.000000000 +0100
@@ -14,6 +14,7 @@
 
 @given(some_object)
 @settings(max_examples=500)
+@example({"": None})
 def test_json5_loads(input_object):
     dumped_json_string = json.dumps(input_object)
     dumped_json5_string = json5.dumps(input_object)
```

Here we have used the [@example](https://hypothesis.readthedocs.io/en/latest/reproducing.html#hypothesis.example) decorator to add a hard-coded example in addition to generated input.

## Libraries
This article has been using the beautiful [Hypothesis](https://hypothesis.readthedocs.io/en/latest/) library for Python. It has a lot of functionality not covered here and nicely written documentation, so be sure to check it out.

Some alternatives for other languages are:

- [fast-check](https://github.com/dubzzz/fast-check): TypeScript
- [FsCheck](https://fscheck.github.io/FsCheck/): .NET
- [jqwik](https://jqwik.net/): Java
- [PropCheck](https://github.com/alfert/propcheck): Elixir
- [PropEr](https://proper-testing.github.io/): Erlang
- [QuickCheck](https://hackage.haskell.org/package/QuickCheck): Haskell
  - There is also a [port of QuickCheck to Rust](https://docs.rs/quickcheck/0.9.2/quickcheck/)


## Follow us
At [Meeshkan](https://meeshkan.com/) we are working on improving how people test systems. Follow us at https://twitter.com/meeshkanml or reach out to us on https://gitter.im/Meeshkan/community!

## Why are you not using property based testing?
Why are you not using property based testing? Interested in seeing another article expanding on the topic? Let us know in the comments!

## Meta: Running the tests
Execute `make` to run the tests (it will setup a `venv` folder and install dependencies there). It will also format the code using [black](https://black.readthedocs.io/en/stable/) and [isort](https://timothycrosley.github.io/isort/) automatically.

# Meta: Where to announce and ask for feedback
- https://hypothesis.readthedocs.io/en/latest/community.html mentions their IRC channel and mailing list.
- https://reddit.com/r/programming and https://reddit.com/r/python perhaps?
- https://news.ycombinator.com/
- https://twitter.com/meeshkanml
- TODO: more.

# Meta: Various resources
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
