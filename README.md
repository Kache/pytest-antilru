# pytest-antilru

[![Build Status](https://travis-ci.com/ipwnponies/pytest-antilru.svg?branch=master)](https://travis-ci.com/ipwnponies/pytest-antilru)
[![Coverage Status](https://img.shields.io/coveralls/github/ipwnponies/pytest-antilru.svg)](https://coveralls.io/github/ipwnponies/pytest-antilru?branch=master)
![license](https://img.shields.io/github/license/ipwnponies/pytest-antilru.svg)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
![supported python versions](https://img.shields.io/pypi/pyversions/pytest-antilru.svg)
![pypi wheels](https://img.shields.io/pypi/wheel/pytest-antilru.svg)
![pypi development status](https://img.shields.io/pypi/status/pytest-antilru.svg)

Caching expensive function calls with `functools.lru_cache` is simple and great performance optimization.
It works so well that it'll even speed up your unit test runs!
Too bad it violated test isolation and caches the wrong values under test conditions, introducing test pollution
(persisted state between test runs).
This package will bust the `lru_cache` between test runs, avoiding test pollution and helping you keep your sanity.

Imagine you mock a network call out and your application ends up caching these mocked results:

```python
def expensive_network_call() -> int:
    # Pretend this is an expensive network call.
    # You want to cache this for performance but you want to run tests with different responses as well.
    return 1


@lru_cache()
def cache_me() -> int:
    return expensive_network_call()
```

Now you have test pollution:

```python
def test_a_run_first() -> None:
    assert cache_me() == 1


def test_b_run_second() -> None:
    # We want to mock the network call for this test case
    with mock.patch.object(sys.modules[__name__], 'expensive_network_call', return_value=2) as mock_network_call:
        assert cache_me() == 2
        assert mock_network_call.called
```

On your next test run, it doesn't matter what you
mock, the results are already cached. Now trying running those two test out-of-order sequence and tell me how it goes.

## Dependencies

Since this is a `pytest` plugin, you need to be using `pytest` to run your tests.

## Compatibility

This project is python 2.7 and python 3.5+ compatible.
It is compatible with pytest 2+.
While we aim to support a wide range of python and pytest combinations, pytest only supports the latest release:
they do not patch older releases to work with newer python versions.
See [tox.ini] for the full envlist of what is being tested.

If you experience issues, please check for compatibility between your python and pytest target versions.
Open an issue once these are verified.

## Installation

Simply install this in the same python environment that `pytest` uses and the rest is magic.

```sh
pip install pytest-antilru
```

## How to test the software

```sh
make test
```

----

## Credits and references

This project was a re-engineering of a similar project a colleague of mine wrote.
That project was not intended to be open-source and rather than go though all the hoops and hurdles to sanitize it,
I've written it from the ground up such that it's kosher to open-source (given that it's such as small project).
