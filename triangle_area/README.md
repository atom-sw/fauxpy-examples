# Triangle Area

You can skip
[Section 1. Project Structure](#1-project-structure)
as FauxPy is totally automated, and
users do not need to know
anything about the project they
are running FauxPy on.

## 1. Project Structure

The current example has two 
packages: [code](code) and [tests](tests).
The code package contains the project's source
code, which are two Python 
modules: [code/equilateral.py](code/equilateral.py)
and [code/isosceles.py](code/isosceles.py).
Each of these modules has one function, which
is buggy. The bug locations are 
marked with comment `bug` within source code, and
the patch for each
bug is in the line following the bug location, 
in the form of a comment.
The function in module `equilateral.py`
computes the area of an equilateral triangle
and the function in module `isosceles.py` computes
the area of an isosceles triangle.

The `tests` package contains the project's
test suite, including two test modules
[tests/test_equilateral.py](tests/test_equilateral.py)
and [tests/test_isosceles.py](tests/test_isosceles.py)
for modules `equilateral.py` and `isosceles.py`,
respectively. Each of these two test modules have two
tests, one failing (i.e., revealing the bug) and
one passing on 
their corresponding modules in package `code`.

## 2. Preparing the Python Environment

We first need to create a 
Python environment for this project, following
the instructions below.

1. Run the following command to create
a Python 3.8 virtual environment.
FauxPy has been test on Python 3.6, 3.7, and 3.8.
But, it should work on other versions as well.

```
python3.8 -m venv env
```

2. Activate environment `env`.

```
source env/bin/activate
```

3. Install FauxPy in environment `env`.

```
pip install fauxpy
```

Keep environment `env` activated for the
rest of this document.

## 3. Locating the Bug in equilateral.py

FauxPy is a Pytest plugin, and thus, running it
is as easy as running Pytest.
So, let's first use Pytest to run all the
tests in package `tests`.

```
python -m pytest tests
```

Running this command ends with the following message,
indicating there are 4 tests in the project, 2 of which
are failing, which is correct.

```
2 failed, 2 passed in 0.07s
```

### 3.1 Running SBFL

Now let's run FauxPy.
FauxPy has only one mandatory command line option
`--src` that takes a package (directory)
or module (`*.py` files) in the project at hand.
Since the source code of our project is in package `code`,
we pass `code` to `--src`.

```
python -m pytest tests --src code
```

The command finishes quickly, ending with
three lists showing up on
the terminal for the three SBFL techniques 
FauxPy supports: Tarantula, Ochiai, and DStar.
By default, FauxPy runs SBFL if the family
is not specified by the user.

The list for Tarantula should look something like
the following. Each row in this list
shows the name of an entity (a line number in package `code`)
along with a number (e.g., 1.1) demonstrating 
its suspiciousness score.

Tarantula was able to find both bugs.
The bug in `equilateral.py` is the second
row in the list, and the bug in `isosceles.py`
is in the fifth row.

``` 
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::13', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::11', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::10', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::8', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::6', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::5', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::10', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::7', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::5', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::11', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::8', 0.1)
```

As you can see, technically, FauxPy can be used
to find multiple bugs at the same time.
However, in such cases,
the output list is noisy.

To make fault localization more effective,
it is better to point FauxPy to a single
bug in every fault localization session.
There are two approaches to do that: 
1. Controlling the test suite
2. Targeting a failing test


*Approach 1: controlling the test suite.* The following 
command runs FauxPy,
using only the tests in `tests/test_equilateral.py`.
Since the failing test in `tests/test_equilateral.py`
is related to only a single bug,
FauxPy focuses only on one bug.

```
python -m pytest tests/test_equilateral.py --src code
```

Tarantula's output list for Approach 1 (controlling the test suite):

```
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::13', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::11', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::10', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::7', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::5', 0.6)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::8', 0.1)
```

*Approach 2: targeting a failing test.* The following
command runs FauxPy, using the whole
test suite `tests`. But this time,
we ask FauxPy to focus only on one failing test
`tests/test_equilateral.py::test_ea_fail`,
which is related to only a single bug.

```
python -m pytest tests --src code --failing-list "[tests/test_equilateral.py::test_ea_fail]"
```

Tarantula's output list for Approach 2 (targeting a failing test):

```
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::13', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::11', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::10', 1.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::7', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::5', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::8', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::6', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::5', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::11', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::10', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::8', 0.1)
```

As you can see, both approaches report the bug in `equilateral.py`
as the second element in the output list, i.e., they demonstrate
the same effectiveness. In fact, a user has to go through three lines
in `equilateral.py` to find the bug as the three lines 13, 11, and 10
in `equilateral.py` share the same and highest suspiciousness score, and thus,
there is no order among them.

### 3.2 Running MBFL

To run MBFL techniques, we must specify the family name using
option `--family`.
If this option is not used, FauxPy runs SBFL by default.
The following command runs the two
MBFL techniques FauxPy supports (Muse and Metallaxis):

```
python -m pytest tests --src code --family mbfl --failing-list "[tests/test_equilateral.py::test_ea_fail]"
```

By running command above, FauxPy generates 32 mutants,
and then, it attempts to kill them. Finally, it reports
two output lists, one for Muse and one for Metallaxis.

Muse's output list is as follows:

```
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::11', 0.09090909090909091)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::10', 0.0)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::7', -0.039660506068057426)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::5', -0.055524708495280385)
```

In Muse's list, the first row (line 11) is the bug in `equilateral.py`.
In fact, Muse assigns the highest suspiciousness score only to 
line 11, indicating the highest effectiveness possible as
the user only has to go through one line in code to find this bug.

### 3.3 Running ST and PS

Running the two other families supported by FauxPy (PS and ST)
result in empty output lists, meaning they are ineffective
at locating the bug in `equilateral.py`.

To run ST, we pass `st` to `--family`:

```
python -m pytest tests --src code --family st --failing-list "[tests/test_equilateral.py::test_ea_fail]"
```

And, to run PS, we pass `ps` to `--family`:

```
python -m pytest tests --src code --family ps --failing-list "[tests/test_equilateral.py::test_ea_fail]"
```

## 4. Locating the Bug in isosceles.py

To locate the bug in `isosceles.py`, we can run the following commands.

To run SBFL techniques, we run the following command.
Note that we changed the argument passed to `--failing-list`
because here we want to locate a different bug.

```
python -m pytest tests --src code --family sbfl --failing-list "[tests/test_isosceles.py::test_ia_crash]"
```

Tarantula's output list:

```
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::8', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::6', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::5', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::10', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::11', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::8', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::7', 0.1)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::5', 0.1)
```

To run the MBFL techniques, we run the following command:

```
python -m pytest tests --src code --family mbfl --failing-list "[tests/test_isosceles.py::test_ia_crash]"
```

Metallaxis's output list:

```
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::10', 0.5)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::6', 0.5)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::8', 0.5)
```

We can run ST and PS simply by replacing `mbfl` with `st` or `ps`
in the previous command. However, since ST and PS only work
with failing tests, to achieve higher
efficiency (with the same effectiveness),
we use Approach 1 (controlling the test suite) detailed
in [Section 3.1 Running SBFL](#31-running-sbfl)
to exclude the passing tests in the test suite.

Run the following command to run ST.
Note that, here, we do not need the `--failing-list`
option since the test suite we are using simply
contains one failing test.

```
python -m pytest tests/test_isosceles.py::test_ia_crash --src code --family st
```

The following list is ST's output produced by FauxPy.
As can be seen here, ST only works at the function granularity.

```
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::height::5::8', 1.0)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::isosceles_area::4::11', 0.5)
```

To run PS, we can replace `st` with `ps` in the
previous command. However, for this bug, PS returns an empty list.

```
python -m pytest tests/test_isosceles.py::test_ia_crash --src code --family ps
```

## 5. Function-level Granularity

All of the examples we have seen so far, runs FauxPy
at the statement-level granularity except for ST, which only works
at the function-level granularity.

FauxPy supports the function-level granularity, as well.
To run any of the previous examples at the function-level
granularity, simply use option `--granularity` by passing
`function` as the argument.
Since the default value for option `--granularity` 
is `statement`, we did not have to use it for
the previous examples.

For instance, we can run the following command to use
FauxPy at the function-level granularity to find the buggy function
in module `isosceles.py`.

```
python -m pytest tests --src code --family sbfl --granularity function --failing-list "[tests/test_isosceles.py::test_ia_crash]"
```

Tarantula's output list:

```
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::isosceles_area::4::11', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/isosceles.py::height::5::8', 0.75625)
('~/fauxpy-examples-dev/triangle_area/code/equilateral.py::equilateral_area::4::13', 0.1)
```

