# Python notes

* [Python language](#Python-language)
* [Python environment, packaging, deployment](#Python-environment-packaging-deployment)
* [Python packages and modules](#Python-packages-and-modules)
* [Python packaging](#Python-packaging)
* [Git manipulations](#Git-manipulations)
* [Github pages](#Github pages)

# Python language

## Comparison and None

Comparison:

* 'is' is identity testing (reference comparison) equivalent to id(a) == id(b)
* '==' is equality testing (content comparison)

Check for non null:

```python
if val is not None:
if val: # Same as above
if val is None:
```

The following is not recommended (because controversies if val is a complex object like pandas data frame):
```python
val != None
```

## Pass arguments

Passing arguments: https://stackoverflow.com/questions/334655/passing-a-dictionary-to-a-function-in-python-as-keyword-parameters

```python
mydict = {'b': 100, 'c': 200}
test(a=3, **mydict)
```

Important is that the dict cannot overwrite the explicit arguments, and the dict cannot contain unspecified (in the function) arguments.	
	
One generic approach is to merge all possible sources of arguments into one dict and then use only this one dict. Also, we need to remove unspecified fields from this dict, for example, if this dict originates from some configuration.

How to merge two dicts: https://stackoverflow.com/questions/38987/how-to-merge-two-dictionaries-in-a-single-expression

Python 2:

```python
z = x.copy()
z.update(y) # which returns None since it mutates z
```

Python 3.5:

```python
z = {**x, **y}  # values from y will replace those from x
```

Delete entry from dict: https://stackoverflow.com/questions/5844672/delete-an-element-from-a-dictionary	

```python
d.pop("key")  # It mutates dict
del d['key']  # Mutates dict
```
python
Copy dictionary (shallow copy):

```python
r = dict(d)
```

## Internal structure of Python objects

Find a function in a module:

```python
import foo
method_to_call = getattr(foo, 'bar')
result = method_to_call()
```

or shorter

```python
import foo
result = getattr(foo, 'bar')()
```

Here you have to know already the module name. 'hasattr' can be used to determine if a function is defined. This version might be better:

```python
getattr(foo, 'bar', lambda: None)
```

Find a function by name:

* 'locals()["myfunction"]()' - use a local symbol table https://docs.python.org/3/library/functions.html#locals
* 'globals()["myfunction"]()' - use a global symbol table http://docs.python.org/library/functions.html#globals

If it is necessary to also import a module:

```python
module = __import__('foo')  # Python 2
module = importlib.import_module  # Python 3
func = getattr(module, 'bar')
func()
```

## Date and time

Definitions:	

* Unix time = POSIX time = UNIX Epoch time = number of seconds elapsed since 01.01.1970 00:00:00 UTC minus leap seconds.
* UTC (Coordinated Universal Time) - no daylight saving time - normally same as GMT but GMT is not precisely defined.

Links:

* Date transformations: https://stackoverflow.com/questions/8777753/converting-datetime-date-to-utc-timestamp-in-python/8778548#8778548

# Python environment, packaging, deployment

## User space

User space is where all packages are stored for only this user. It is analogous to the global dir but only for this user.

Install into user space: 

```python
pip install --user package-name
```

User-base binary directory is needed and must be in PATH. how to get user-base dir:

* Linux: "python -m site --user-base" (typically ~/.local) - and then add /bin to the end
* Windows: "python -m site --user-site" (typically AppData\Roaming\Python\Python36\site-packages) - and replace site-packages with Scripts

## virtualenv
	
virtualenv is a tool to create isolated Python environments. virtualenv creates a folder which contains all the necessary executables to use the packages that a Python project would need.

```
pip install virtualenv - install the tool
```

Create a virtualenv for a project:

```
cd my_project_folder 
virtualenv my_project
```

It will be in the current directory (where the project is) with Python executables and copy of pip (to install other packages). Exclude this folder from source control (or use standard name for all projects like 'env').

Use any python interpreter of your choice (or we can use envvar VIRTUALENVWRAPPER_PYTHON as a global parameter):

```
virtualenv -p /usr/bin/python2.7 my_project - 
```

Activate the current virtualenv (otherwise it will not be used):

```
my_project/bin/activate
```

From now on, all packages installed using pip will be placed in this virtualenv folder (not global).

Deactivate the current virtualenv (switch to system default):

```
deactivate
```

Delete the folder to delete the virtualenv

Freeze the current state of packages etc. in a file:

```
pip freeze > requirements.txt
```

Recreate the environment:

```
pip install -r requirements.txt"
```

## pipenv

Install pipenv in user space:
```
pip install --user pipenv 
```
Install globally:
```
pip install pipenv
```

Pipenv manages dependencies on a per-project basis. So we need to change into the project dir and do installation from it:

```
pipenv install some-package
```

It will install this package and update Pipfile (which tracks your dependencies).

For each project it will create locally (for the user): a separate virtualenv (in \.virtualenvs folder) and a separate Pipfile (in project folder)

```
pipenv install # No package. it will use requirements.txt and install the packages.
```

Links:
* https://docs.pipenv.org/
* http://docs.python-guide.org/en/latest/dev/virtualenvs/


# Python packages and modules

Terms and principles:
* Module is a file
* Package is a folder with modules inside (for Python 2 it must have `__init___` in it)
* It is possible to import both packages and modules. Importing a package means importing `__init___.py` as a module
* `import abc` imports either package or module
* `from abc import xyz` imports module, subpackage or class/function
* `import abc as other_name` renames the imported resource. It must be then names using new name.

Import should work without changes in the following cases:
* Execute a local module from IDE (debugging), e.g., using its main function or from command line in the same directory: `module3.py`
* Execute a local modeul from the project root by specify its location in the package structure like `module1/module2/module3.py`
* Import of arbitrary module in the hierarchy from within another internal module in this hierarchy

What is important:
* The directory the program has been started
* The project root
* What is declared in `__init___.py`
* How to import: import package/module [as name], from package/module import name or *
* Relative package/module names: module1.module2.module3, ...module1.module2

How import works. When `import abc` is executed:
* Python will look up the `abc` name in `sys.modules` which is a cache of all previously imported modules
* Then it searches in a list of built-in modules
* Then it searches in a list defined in `sys.path`. It typcially includes the current directory from which the input script was run or the current directory if the interpreter is being run interactively, as well as paths in the PYTHONPATH (typically same as PATH)
  * This is why we add a path with our module: `sys.path.append(r'C:\Users\john')`
* Otherwise, it throws `ModuleNotFoundError`
* Once the module was imported, its path is here: `abc.__file__`

Absolute vs. relative imports:
* An *absolute import* specifies the resource to be imported using its full path from the project’s root folder. This is somewhat similar to its file path, but we use a dot (.) instead of a slash (/).
  * Pros: import path does not depend on the location of this file and the import statement itself can be copied to other files
  * Pros: Each element has a unique location
  * PEP 8 explicitly recommends absolute imports
  * Cons: paths can be quite long
    * Q: Can we solve this problem by using `as name` for long import paths?
  * Q: How project root is defined in code and in environmnet?
* A relative import specifies the resource to be imported relative to the current location—that is, the location where the import statement is
  * Implicit relative imports have been deprecated in Python 3, so I won’t be covering them here
  * They start from dot (in contrast to absolute imports)
  * Single dot means current location 
  * `from . import class1` imports from the current package (effectively from `__init___`)

* Q: When to use `import abs` and where to use `from something import abc`
* Q: How `sys.path` influences absolute and relative imports?
* Q: HOw environment variables like `PYTHONPATH` influence absolute and relative imports?
  * PyCharm has checkboxes: add content roots to `PYTHONPATH`, and add source roots to `PYTHONPATH`
  * Maybe we could do it programmatically in our scripts (instead of or in addition to `sys.path` modification)?
* Q: How cwd (program start) influences imports
* Q: How main program properties influence imports (like `__name__` and similar properties)?

# Python packaging

### Links

* https://blog.jetbrains.com/pycharm/2017/05/how-to-publish-your-package-on-pypi/
* https://www.knowledgehut.com/blog/programming/how-to-publish-python-package-to-pypi

### Sequence of submission to PYPI using twine

* `python -m pip install --upgrade setuptools wheel` Upgrade packages
* `python -m pip install --upgrade twine` Upgrade the submission package. Use --user option if necessary
* `python setup.py sdist bdist_wheel` Generate package. Ensure that only necessary folders are used (remove them). Clean dist folder before. 
Source dist is needed by pypi only as a fallback. There can be many build distributions (wheel) for different platforms but source dist is only one. 
Both archives will be uploaded to pypi and visible/downloadable from the browser.
* `twine upload --repository-url https://test.pypi.org/legacy/ dist/* - test` Test the submission. Separate login is needed.
* `twine upload dist/*` Only files from the dist folder will be used. Login/password will be asked. The pypi user has to own the project (with the name defined in setup.py)
* Check that the package has been uploaded here: https://pypi.org/search/

# Git manipulations

### Change authors of commits

#### Approach 1 (git-filter-repo)

This utility is one Python file but it also can be installed using pip:
```
pip install git-filter-repo
```

Create a file `.mailmap`  with the mapping of old authros and new authros which has lines like this:
```
New Name <new_email@users.noreply.github.com> Old Name
```
Other options can be found here: 

Execute:
```
git-filter-repo --mailmap ../.mailmap 
```

The repo now will also include a mapping from new hashes to old hashes which, particularly, will show up in the log next to each commit message. These mappings can be removed manually:
* delete file `.git/info/refs` which stores a two-column mapping from new hashes to old hashes
* from file `.git/packed-refs` delete all lines which map to refs/replace/* and hence we forget how new hashes map to old hashes
* delete `ORIG_HEAD`

Another alternative (not tested) is to play with options like
```
--replace-refs {delete-no-add, delete-and-add, update-no-add, update-or-add, update-and-add}
```

For complex renamings, call back functions in Python can be used:
```
git-filter-repo --name-callback 'return name.replace(b"OldName", b"NewName")' --email-callback 'return email.replace(b"old@email.com", b"new@email.com")'
```

Links:
* https://stackoverflow.com/questions/59590204/changing-git-author-info-on-all-commits-worked-on-one-of-my-repos-but-not-the-ot
* https://github.com/newren/git-filter-repo

After the repo has been re-written, it can be pushed (see another question).

### How to push local repo to an existing remote repo

Here we assume that the local repo was not checked out from the remote one. For example, we might create the local repo by cleaning up some repo. A remote repo was created as a new repo. A typical scenario is where we want to move some repo by cleaning it up from some previous commits.

One approach is to check out, clean old commits, and then push it back. Here the question is whether the server will overwrite its old state.

Another option is to check out, clean by ensuring that it does not contain anything we wanted to delete, and then push it to a completely new repository (by thus ensuring that the server does not store anything from the old state). Note that the local repository is also cleaned from any references to its (old) origin. For the server it looks like we create a new repo and then push a whole history from some local repo with unknown history (we might maintain it only locally).

```
# Push an existing repository from the command line (github repo must exist)
git remote add origin https://github.com/c0ldlimit/vimcolors.git
git push -u origin master
```
The first command will change config. Here we attach origin and then push it.

Yet, the remote and local repositories might have some unsync state in terms of commits which results in this error message when attempting to push:

    Updates were rejected because the remote contains work that you do not have locally. You may want to first integrate the remote changes (git pull)

One solution is to use `--force` flag (instead of `-u` or addition to):
```
git push origin master --force
git push -u --force origin master
```

Links:
* https://gist.github.com/c0ldlimit/4089101
* https://blog.plover.com/prog/git-ff-error.html

### Remove some (sensitive) commits from git history

Remove the old files
```
git gc --aggressive --prune=all
or
git gc --auto
```

Links:
* https://stackoverflow.com/questions/872565/remove-sensitive-files-and-their-commits-from-git-history

### Empty remote repository

Create a completely new repo and then push it to remote:
```
git init
git commit -S -m "Initial commit" --allow-empty  # for empty commit
git remote add origin https://github.com/asavinov/test.git
git push -u --force origin master
or
git push --mirror --force
```

The remote will still store old history but it will be removed after garbage collection (see another question)

Another option is to delete the whole branch:
```
git checkout temp  # we cannot delete current branch so we create a temporary
git branch -D master
git push origin :master
```
We can delete remote branch which is not default, so change it in setting. Directly remote branch:
```
git push origin --delete master
```

# Github pages

### Types of github pages

* User or organizations. They do not have repositories. Therefore, the content for the page is supposed to be located in a repository with the same name: `username.github.io`. The content of this repository (with a special name) will be automatically shown here https://username.github.io
* Repository. The content is supposed to be stored in some branch of the repository and folder which are specified in the settings. After that, the page is available here: http://username.github.io/repository
* Repository. An alternative approach would be to create a dedicated repository for documentation, for example, http://username.github.io/repository_docs. Then it is necessary to configure a job for checking out the source code of documentation from the project repository and then committing it (after compilation) to the documentation repository.

### Deploy documentation to github pages

In order to only build documentation, use the corresponding action:
* https://github.com/marketplace/actions/sphinx-build

To deploy and host documentation on github pages, it is necessary to store somewhere in the repository its (compiled) sources (html pages). There are two cases:
* For any (public) repository, we can specify where the content of its github pages is located by choosing branch and folder (in settings). Then this content will be shown at the url `username.github.io/reponame`.
* Dedicated repository for sources. For example, special repo name `asavinov.github.io` stores content for the whole account. 
* Is it possible to re-use the special repo name (for the whole account) for docs for all projects in this account? Indeed, it should be possible to push content to certain folders only like `/repo1` and `/repo2` from actions executed in these separate projects.

If we want to store compiled content in a dedicated repo then it will have a name different from the main project name like `username.github.io/reponame_docs`. If we want to have same name for documentation as the project name then we need to store the content in some branch. Typically, documentation to be shown in github pages is stored in the branch `gh-pabes`.

Uploading compiled content to a repo (branch) is performed using github action.
