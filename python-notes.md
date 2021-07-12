# ---
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

# ---
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

# ---
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

# ---
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
