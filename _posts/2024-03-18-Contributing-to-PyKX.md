# Contributing to PyKX
## Introduction
Contributing to an Open Source project can be daunting at first, but if you are proficient in both Python and Q, there is a relatively simple way to contribute to OSS.

### Enter PyKX
[PyKX]() is a library tailored towards Python users that aims to bring them closer to the KDB+/Q environment by enabling the use of Q from a Python environment and viceversa. It also provides conversion between Python and Q types, making migrations from Python to q as seamless as possible. We dedicated two fantastic articles that went into a lot more detail about this migration process [here]() and [here](). More specifically, we laid down a simple path that users should follow if they wished to perform a similar kind of migration.

However, that's not the topic of today's blog post. As we have already established, PyKX is the perfect gateway into Open Source development for those with knowledge of both Python and Q, and in this article we will give you a step by step guide on how we  implemented a simple function and contributed to this fantastic library.

Since we had some previous knowledge on [pandas](), we decided to tackle the pandas API that the PyKX team had been working on for a while. From our previous efforts with this API, we located a few functions that were in need of an implementation, so once we had our goals set we got down to business!

### Setting up the development environment
The very first thing you should do after setting up your environment should be to fork [KxSystems/pykx]() into your own repository where you have permissions to create branches and do some actual work.

The first roadblock for us however was the environment setup, since we hadn't worked on a Python library before. If you land on their [GitHub repository]() like we did, chances are you will take a look at their README. This is a really nice introduction to what PyKX is and their goals with the library, but for this section we will focus on the [`Building from Source`](https://github.com/github/pages-gem/issues/752) section.

Here, you should follow their instructions carefully. In our case, we decided to install on Linux, since it is our platform of choice for these kind of projects, but installations steps are available for those using Windows as well!

As a side note, we were not able to setup the environment on Fedora Linux, but we were successful on Ubuntu both on bare metal and on [WSL]().

The quick by step goes as follows:

 1. Install the build dependencies.
 2. Create the Python virtual environment (venv from now on) and activate it.
 3. Run `pip3 install -U '.[all]'` to build the library and make it available on your venv.

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

---

_**Note**_:

At the end of the post we will provide some useful commands, as well as some general tips to developing for this library.

---

**Congratulations!** You have now set your development environment up and are ready to start writing some code!

### Useful tools
In our case, we used mostly VSCode when developing for this library, but feel free to use your editor of choice.

Inside your venv, be sure to have Jupyter installed to have quick access to your function after the build. This is particularly usefull when developing some tests. A q kernel for Jupyter may also be handy when debugging your q code.

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

Those of you proficient on pandas and Q might have noticed that `count each` doesn't do exactly the same as `df.count()` on pandas. The later doesn't count the null values whereas the q version does. This is intentional in this case, and later on we'll see how we take care of those edge values.

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

Well, all of them contain code for the pandas API, of course, but where we place it is up to our decision. Is it related to indexing? Then write your function inside `pandas_indexing.py`. Is it a special type of join? Then `pandas_merge.py` is most likely where you want to write it.

In our case, since `count` is a general function, we can place it under `pandas_meta.py`, which contains a plethora of general purpose functions. Inside that file, try to write your implementation around similar functions.

 ## Getting familiar with the syntax

As with any new environment you need to work on, it may take a while to get used to it. I would personally look around the different files and try to take mental notes (or regular notes, I won't judge you) on how the functions are implemented, which patterns repeat themselves, which don't, etc... 

If you have done just that for the `pandas_meta.py` file, you may have some questions. What does `@convert_result` and `@api_return` do? Why do most functions use `preparse_computations` on the first line? Let's try to answer those questions.

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


## Linting

## Opening a Pull Request

## Conclusions
