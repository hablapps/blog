# Contributing to PyKX

## Introduction

Contributing to an Open Source project can be daunting at first glance, but if you have some know-how in both Python and Q, you have an opportunity to become a contributor in KX's PyKX library.

### Enter PyKX

[PyKX]() is a library tailored towards Python users that aims to bring them closer to the KDB+/Q environment by enabling the use of Q from a Python environment and vice-versa. It also provides conversion between Python and Q types, making migrations from Python to Q as seamless as possible. We dedicated two fantastic articles that went into a lot more detail about this migration process [here]() and [here](). More specifically, we laid down a simple path that users should follow if they wished to perform a similar kind of migration.

However, that's not the topic of today's blog post. As we have already established, PyKX is the perfect gateway into Open Source development for those with knowledge of both Python and Q, and in this article we will give you a step by step guide on how we  implemented a simple function and contributed to this fantastic library.

Since we had some previous knowledge on [pandas](), we decided to tackle the pandas API that the PyKX team had been working on for a while. From our previous efforts with this API, we located a few functions that were in need of an implementation, so once we had our goals set we got down to business!

### Setting up the development environment

The very first thing you should do after setting up your environment should be to fork [KxSystems/pykx]() into your own repository where you have permissions to create branches and do some actual work.

If you land on their [GitHub repository]() like we did, chances are you will take a look at their README. This is a really nice introduction to what PyKX is and their goals with the library, but for this section we will focus on the [`Building from Source`](https://github.com/github/pages-gem/issues/752) section.Here, you should follow their instructions carefully. In our case, we decided to install on Linux, since it is our platform of choice for these kind of projects, but Windows and OS X are viable options as well!

Be sure to take a look at their `DEVELOPING.md` file for some development guidelines. This file contains the team's code styling preferences, so be sure to stick to them.

The quick step by step goes as follows:

 1. Install the build dependencies.
 2. Create the Python virtual environment (venv from now on) and activate it.
 3. (Optional) Run `pip3 install -U '.[all]'` to build the library and make it available on your venv.

Be aware that the last step may take a few minutes depending on your hardware.

---

_**Note**_:

We also tried to use [Pip's Editable Builds](https://setuptools.pypa.io/en/latest/userguide/development_mode.html), but since this library relies on Cython for some parts, we were unsuccessful on trying to make it work. If this was available, time spent on builds would be drastically reduced.

---

In order to be able to run some tests (which will be required for the contribution), you need to also run these commands (on Linux), as the documentation states:

```sh
export PATH="$PATH:/location/of/your/q/l64" # q must be on PATH for tests
```

```sh
export QHOME=/location/of/your/q #q needs QHOME available
```

With that done, you should be able to run the tests for the whole project with the following command:

```sh
python -m pytest -vvv -n 0 --no-cov --junitxml=report.xml
```

Again, be aware that this command may take several minutes to complete, since it's running _every_ test. Since this is completely optional at this point, feel free to skip it. Later on we will provide a command to run a single test, which makes the whole development experience much more streamlined.

**Congratulations!** You have now set your development environment up and are ready to start writing some code!

### Useful tools

 * As for the IDE / text editor, feel free to use your Python editor of choice.

 * Inside your venv, be sure to have Jupyter installed to have quick access to your function after the build is completed. This is particularly useful when developing tests. A [Q kernel for Jupyter]() may also be handy when debugging your Q code. As an alternative, a [VSCode plugin]() that makes you able to run Q code also exists.

---

_**Note**_:

To install Jupyter inside your venv, run the following command after activating it:

```sh
pip3 install jupyter
```

Or

```sh
pip3 install jupyterlab
```

If you would rather use the lab environment.

---

## Development

With all of that done, we will focus on developing a simple function for the pandas API: `count`.

After taking a decision on the function we will be developing, since we are making a contribution to the pandas API, we should take a look at what that function actually does. In this particular case, [`count`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.count.html) returns a dictionary of sorts with the columns as the key and the count for each column as the value:

```python
> df = pd.DataFrame({"Person":
                   ["John", "Myla", "Lewis", "John", "Myla"],
                   "Age": [24., np.nan, 21., 33, 26],
                   "Single": [False, True, True, True, False]})
> df
```
```
   Person   Age  Single
0    John  24.0   False
1    Myla   NaN    True
2   Lewis  21.0    True
3    John  33.0    True
4    Myla  26.0   False
```

```python
> df.count()
```
```
Person    5
Age       4
Single    5
dtype: int64
```

The implementation should be easy on q: we can just `count` on each column and we should get the result we want!

---

_**Note**_:

Those of you proficient on pandas and Q might have noticed that `count each` doesn't do exactly the same as `df.count()` on pandas. The later doesn't count the null values whereas the Q version does. This is intentional in this case, and later on we'll see how we take care of those edge values.

```q
q)t
a b
---
a 0
b 1
  2
q)count each value flip t
3 3
```

---

Now that the underlying Q code is clear, we can focus on bringing the functionality to PyKX. For that, we need to know _where_ to place our function and _how_ to make it available.

PyKX's codebase is quite extensive, so we need to make sense of it. We can assume the source code is inside the `src` directory, and that the pandas API code should be inside the `pandas_api` subdirectory, but there are quite a few Python files inside so... Where do we put our implementation?

INSERT DIRECTORY TREE IMAGE HERE

Well, all of them contain code for the pandas API, of course, but where we place it is up to the "family" of functions it belongs to. Is it related to indexing? Then write your function inside `pandas_indexing.py`. Is it a special type of join? Then `pandas_merge.py` is most likely where you want to write it.

In our case, since `count` is a general function, we can place it under `pandas_meta.py`, which contains a plethora of general purpose functions. Inside that file, try to write your implementation around similar functions so you can them as reference.

 ## Getting familiar with the syntax

As with any new environment you need to work on, it may take a while to get used to it. I would personally look around the different files and try to take mental notes (or regular notes, I won't judge you) on how the functions are implemented, which patterns repeat themselves, which don't, etc... 

If you have done just that for the `pandas_meta.py` file, you may have some questions. What does `@convert_result` and `@api_return` do? Why do so many functions use `preparse_computations` on the first line? Let's try to answer those questions.

Both `@convert_result` and `@api_return` are [decorators](https://peps.python.org/pep-0318/). What that roughly means is that they are able to perform some computation before executing the function and after the fact. In this case, `@convert_result` is defined on that very same file, and does the following:

 1. Runs the "child" function (in this case, `count`).
 2. It builds a dictionary with the functions return values.

So, with this in mind, we know that what we would want to return in our child function if we were using this decorator would be a list of values and the column names so that we can build a dictionary, and that's exactly what we need to return for `count`!

---

_**Note**_

The `@api_return` decorator is used when we want to return a table-shaped object. For example, if we wanted to round all the values from a table or filter it, we would need to use that decorator instead of `@convert_result`. As an exercise, try to find where this decorator is defined and take a look at its implementation to get a better understanding of how and when to use it.

---

## Implementing the function

Next up, I'm going to show you our accepted implementation of `count` and go line by line explaining what's going on. The code looks like this:

```python
@convert_result
def count(self, axis=0, numeric_only=False):
    res, cols = preparse_computations(self, axis, True, numeric_only)
    return (q('count each', res), cols)
```

As you can see, there are a few things going on but the implementation itself has a very small footprint. If we ignore the third line, we can see that we are indeed using the `@convert_result` decorator, so we need to return the dictionary values, which will be the count on each column and the keys which, in this case (and most), will be the column names.

The second line is the function declaration and here we try to stick to [pandas' interface]() so that integration with existing codebases is as seamless as possible. In this case, we need to define the `axis` and `numeric_only` parameters, with their respective default values.

On the last line of code we actually perform the counting on each of our columns' values. If you are familiar with PyKX, you most likely have used `pykx.q` to run Q code from Python, and this is exactly what is going on here. We are running `count each` on our `res` value.

But wait a minute, what does `res` contain? Well, that's a good question. In order to understand that, first we need to take a look at what `preparse_computation` does.

If you have ever worked with pandas, you might have noticed that many methods share a very similar interface: `axis`, `skipna` and `numeric_only`. These parameters, among others, dictate how those methods handle the data. Since they are so common, the PyKX team has developed the `preparse_computation` function to avoid repeating code. I encourage you to take a look at what it does exactly, but in rough terms:

 1. Gets the table's values and column names.
 2. Depending on the parameters, it transforms the column values according to them.
 3. It returns a list containing the column values (so, a list of lists) and the column names.

With this in mind, we can now fully understand the implementation we wrote. In this case, the `preparse_computation` takes the `axis` and `numeric_only` values straight from the parameters and for the `skipna` value we pass a `True`, meaning that if we find a null value, it should be dropped. This last hardcoded parameter allows our `count` to behave exactly as the one found on the pandas library!

## Testing

Now that we have developed our functions, we would want to write some unit tests for it to make the functionality resilient to code changes. For the pandas API, they are coupled together in the `tests/test_pandas_api.py` file. We can see that in that same directory we have different `test_pandas_*.py` files which contain tests for specific functionality, but most tests for the `pandas_meta.py` file are held here.

Since we are mimicking functionality on the function we developed, our tests should reinforce that fact. For `count` we should test:

 * Counting on a table with null values.
 * Counting on a table with non-numeric values.

At this point, we could write some code on the test file and hope for the best, but I personally recommend installing Jupyter inside the venv for the project so that whenever we build our library, it becomes available from a Jupyter notebook where we can start building up our test cases. It may look a bit overkill, but for larger functions with many more moving parts, the ability to quickly prototype your test cases could be crucial.

However you may choose to develop your tests, our input for this function could look something like this:


```python
tab = q('([] k1: 0n 2 0n 2 0n ; k2: (`a;`;`b;`;`c))')
df = tab.pd()
```

Where we take care of both of our testing cases and make the table available for using both pandas and PyKX functions.

As for the assertions, we are going to take advantage of PyKX's ability to convert between PyKX objects and Python ones to make sure that our function returns the expected values.

A simple assertion could look something like this:

```python
qcount = tab.count().py()
pcount = df.count()

assert int(qcount["k1"]) == int(pcount["k1"])
assert int(qcount["k2"]) == 3
```

First off we want to run `count` on both the PyKX table and the pandas DataFrame and since the result of our `count` function is a Q dictionary, we can use `.py()` to convert it to a Python one and then perform the actual assertions after that.

After this set of assertions, we should test our `count` function with different parameters:

```python
qcount = tab.count(axis=1).py()
pcount = df.count(axis=1)

assert int(qcount[0]) == int(pcount[0])
assert int(qcount[1]) == 1
```

```python
qcount = tab.count(numeric_only=True).py()
pcount = df.count(numeric_only=True)

assert int(qcount["k1"]) == int(pcount["k1"])
```

After everything has been taken into consideration, we would want to actually run our test. To do that, we can execute the following command:

```sh
TODO
```

If we wanted to run all tests on a file, we can do it with the following command:

```sh
TODO
```

If all our test cases are well designed and have passed, we should be pretty much good to go! Now we only need to use `pflake8` to lint our code, write some documentation and open a pull request.

### Linting

As mentioned on the `README` file, PyKX should follow the pep8 conventions that the default `pflake8` enforces. Simply running
`pflake8 {path/to/file}` should tell us if our code follows said rules. In our case, we would have to run it for both the `pandas_meta.py` and `test_pandas_api.py` files. Once the output of `pflake8` shows no warnings, we can move on to documenting our new function.

## Documenting our code

In the case of our `count` function, the documentation should be written on the `docs/user-guide/advanced/Pandas_API.ipynb` notebook. Locate the function "family", take a look at the documentation's structure and use it as a reference. Be sure to provide the function's interface and some examples of use.

## Opening a Pull Request

As with most open source projects, code additions should be handled through pull requests. If you are not familiar with pull requests, you can take a look at this [guide]() written by GitHub.

Open an issue following its template located at `TODO:foo` and then open a pull request, linking it with the issue and following the template located at `TODO:bar`.

## Conclusions and general tips

Hopefully we have encouraged you to make your own contribution to PyKX and help grow this fantastic library. After having explained the whole process, here are some quick tips that we found useful when developing our contributions:

 * Try to blend your code as much as possible with the existing codebase.
 * Follow their programming philosophy as stated on the `DEVELOPING.md` file.
 * Write thorough tests. They can help you identify "holes" in your logic.
 * Try to learn as much as possible in the process!