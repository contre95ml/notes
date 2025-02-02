# Python Solutions

## Clear cache one-liner

```sh
find . | grep -E "(_pycache_|\.pyc|\.pyo$)" | xargs rm -rf
```

*Source :https://stackoverflow.com/questions/28991015/python3-project-remove-pycache-folders-and-pyc-files/46822695*

## Colors in stdout

This somewhat depends on what platform you are on. The most common way to do this is by printing ANSI escape sequences. For a simple example, here's some python code from the [blender build scripts](https://svn.blender.org/svnroot/bf-blender/trunk/blender/build_files/scons/tools/bcolors.py):

```python
class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'
```

and then:

```python
print(f"{bcolors.WARNING}Warning: No active frommets remain. Continue?{bcolors.ENDC}")
```

## if _ _name _ _ == “ _ _ main _ _ ”: ?

[Source from Stackoverflow](https://stackoverflow.com/questions/419163/what-does-if-name-main-do)

Whenever the Python interpreter reads a source file, it does two things:

- it sets a few special variables like `__name__`, and then
- it executes all of the code found in the file.

Let's see how this works and how it relates to your question about the `__name__` checks we always see in Python scripts.

**Code Sample**

Let's use a slightly different code sample to explore how imports and scripts work. Suppose the following is in a file called `foo.py`.

```python
# Suppose this is foo.py.

print("before import")
import math

print("before functionA")
def functionA():
    print("Function A")

print("before functionB")
def functionB():
    print("Function B {}".format(math.sqrt(100)))

print("before __name__ guard")
if __name__ == '__main__':
    functionA()
    functionB()
print("after __name__ guard")
```

**Special Variables**

When the Python interpeter reads a source file, it first defines a few special variables. In this case, we care about the `__name__` variable.

**When Your Module Is the Main Program**

If you are running your module (the source file) as the main program, e.g.

```bash
python foo.py
```

the interpreter will assign the hard-coded string `"__main__"` to the `__name__` variable, i.e.

```python
# It's as if the interpreter inserts this at the top
# of your module when run as the main program.
__name__ = "__main__" 
```

**When Your Module Is Imported By Another**

On the other hand, suppose some other module is the main program and it imports your module. This means there's a statement like this in the main program, or in some other module the main program imports:

```python
# Suppose this is in some other main program.
import foo
```

The interpreter will search for your `foo.py` file (along with searching for a few other variants), and prior to executing that module, it will assign the name `"foo"` from the import statement to the `__name__` variable, i.e.

```python
# It's as if the interpreter inserts this at the top
# of your module when it's imported from another module.
__name__ = "foo"
```

**Executing the Module's Code**

After the special variables are set up, the interpreter executes all the code in the module, one statement at a time. You may want to open another window on the side with the code sample so you can follow along with this explanation.

**Always**

1. It prints the string `"before import"` (without quotes).
2. It loads the `math` module and assigns it to a variable called `math`. This is equivalent to replacing `import math` with the following (note that `__import__` is a low-level function in Python that takes a string and triggers the actual import):

```python
# Find and load a module given its string name, "math",
# then assign it to a local variable called math.
math = __import__("math")
```

1. It prints the string `"before functionA"`.
2. It executes the `def` block, creating a function object, then assigning that function object to a variable called `functionA`.
3. It prints the string `"before functionB"`.
4. It executes the second `def` block, creating another function object, then assigning it to a variable called `functionB`.
5. It prints the string `"before __name__ guard"`.

**Only When Your Module Is the Main Program**

1. If your module is the main program, then it will see that `__name__` was indeed set to `"__main__"` and it calls the two functions, printing the strings `"Function A"` and `"Function B 10.0"`.

**Only When Your Module Is Imported by Another**

1. (**instead**) If your module is not the main program but was imported by another one, then `__name__` will be `"foo"`, not `"__main__"`, and it'll skip the body of the `if` statement.

**Always**

1. It will print the string `"after __name__ guard"` in both situations.

***Summary\***

In summary, here's what'd be printed in the two cases:

```python
# What gets printed if foo is the main program
before import
before functionA
before functionB
before __name__ guard
Function A
Function B 10.0
after __name__ guard
# What gets printed if foo is imported as a regular module
before import
before functionA
before functionB
before __name__ guard
after __name__ guard
```

**Why Does It Work This Way?**

You might naturally wonder why anybody would want this. Well, sometimes you want to write a `.py` file that can be both used by other programs and/or modules as a module, and can also be run as the main program itself. Examples:

- Your module is a library, but you want to have a script mode where it runs some unit tests or a demo.
- Your module is only used as a main program, but it has some unit tests, and the testing framework works by importing `.py` files like your script and running special test functions. You don't want it to try running the script just because it's importing the module.
- Your module is mostly used as a main program, but it also provides a programmer-friendly API for advanced users.

Beyond those examples, it's elegant that running a script in Python is just setting up a few magic variables and importing the script. "Running" the script is a side effect of importing the script's module.

**Food for Thought**

- Question: Can I have multiple `__name__` checking blocks? Answer: it's strange to do so, but the language won't stop you.
- Suppose the following is in `foo2.py`. What happens if you say `python foo2.py` on the command-line? Why?

```python
# Suppose this is foo2.py.

def functionA():
    print("a1")
    from foo2 import functionB
    print("a2")
    functionB()
    print("a3")

def functionB():
    print("b")

print("t1")
if __name__ == "__main__":
    print("m1")
    functionA()
    print("m2")
print("t2")
```

- Now, figure out what will happen if you remove the `__name__` check in `foo3.py`:

```python
# Suppose this is foo3.py.

def functionA():
    print("a1")
    from foo3 import functionB
    print("a2")
    functionB()
    print("a3")

def functionB():
    print("b")

print("t1")
print("m1")
functionA()
print("m2")
print("t2")
```

- What will this do when used as a script? When imported as a module?

```python
# Suppose this is in foo4.py
__name__ = "__main__"

def bar():
    print("bar")

print("before __name__ guard")
if __name__ == "__main__":
    bar()
print("after __name__ guard")
```

