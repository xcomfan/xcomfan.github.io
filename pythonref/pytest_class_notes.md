# Intro

* What make a good framework?
    * Fun
    * Fast
    * Minimally complex
    * Actionable reporting


* Popular Python testing frameworks
    * Unittest - Bare bones build into python by default.
    * Nose - built on top of unittest extends some of the missing bits.
    * Behave - 
    * Robot
    * Pytest - Compatible with unit test so you can still run your unit tests.  The one this course will cover.

# Environment Setup

* You just have a virutal environment at `/home/boris/udemy_pytest_course/framework_venv/bin` in your WSL.

* Run `pip install pytest` in your venv to install pytest

* just call `pytest` or `pytest -v` to run your tests.

* moudule and functions that test need to start with test_

* You can manually create a `pytest.ini` file to configure pytest.  

* "test search" is the logic of how the tests you write are found.  What the modules should be named, the test function and the test classes.

# Marking tests

```
from pytest import mark

@mark.engine
def test_engine_functions as expected():
    assert True
```
Then you can run `pytest -m engine` to just run engine test instead of running all of the tests.

You can also have two markers as in example below.

```
from pytest import mark

@mark.smoke
@mark.engine
def test_engine_functions as expected():
    assert True
```

Markers are basically just tags.

To run test for multiples tags/markers you can do something like

`pytest -m "body or engine"` or `pytest -m "body and engine"`

Also a valid option for everything but a certain marker.

`pytest -m "not entertainment"`

In the scenario that you have all tests ina module as body tests instead of marking them all as @mark.body you can put them in  a class as in the example below.

```
@mark.body
class BodyTests:
  @mark.door
  def test_body_functions_as_expected(self):
    assert True

  def test_bumper(self):
    assert True
```

To define the markers for the project you can make an entry in your pytest.ini file similar to the one below.

```
marker =
    smoke: All critical smoke tests
    body: All car body tests
    entertainment: nav system tests
```

You will then see them when you run the command pytest --markers


# Fixtures

- Any fixture you create in `conftest.py` becomes available in any file and directory from where you created the fixture down where you are writing test cases and you don't actually need to import them.

# Display test runs

- You can `pip install pytest-html` and then run your test with `pytest --html="results.html"` to generate a nice report that can be shown to people.  You can also use this library to get an XMl output that Jenkins or other tools can parse rand give you a nice organized display of your test runs.

# Customize test runs with command line and config files

You can create a config fixture that has al of your environment specific configuration values and you can pass that Config class as an argument to your tests so they can have the necessary environment settings.

# Handling spikes and expected failures

## To skip a testa but mark that its being skipped

```
from pytest import mark
@mark.skip or @mark.skip(reason="Your reason here")
def this_is_my_test(some_fixture):
...
```

- You can use an xfail marker if you expect the test to fail.  This is kind of like asserting false but test failing means OK.  Kind of a possible edge case that may be handy.

## Parametarization

- Parametrization means running the same test with different inputs without copying that test many times.

```
from pytest import mark

@mark.parametrize('tv_brand',[
    ('Samsung'),
    ('Sony'),
    ('Vizio')
])
def test_television_turns_on(tv_brand):
    print(f"{tv_brand} turns on)
```

- The above example will run 
- An issue with example above is that your data is coupled to the test.  You can avoid this issue with the approach below...

when defining your fixture you can pass a parameter argument, and the test will run for each parameter.

```
@fixture(params=[webdriver.Chrome, webdriver.Firefox, webdriver.Edge])
def browser(request):
    driver = request.param
    drvr = driver()
    yield drvr
    drvr.quit()
```

# Running your tests in parallel

You can pip install the pytest-xdist package and call pytest -n # of cores which will run the number of threads yo specify and make things run faster.

- for threading to work you need to write your tets to be isolated so that them running in parallel does not make them interfere with each other.

# Unit Testing

- If you have your test cases in a module you may have issues with Python pathing picking up that modules.  One way to fix this is to do a dummy pip install of your current work area.  This can be done with the command below which will install current directory as a package.

`pip install -e .`

# Tox (Tool that helps you separate concerns between library you are testing and your tests)

- see the docs and re-watch the video for this one.  Too much info to get all context here.
