# git-vendor-mirror Workflow Specification

`git-vendor-mirror` is basically a complicated shell script (implemented not-in-shell) that orchestrates a specifically defined Git workflow, provides standard commit messages, and (in doing so) makes it easier to navigate what's going on with the third-party source code you're using.

This workflow could be replicated by judicious use of various `git` commands in the correct incantation, and really, `git-vendor-mirror` is just a convenience for users.

This document attempts to provide the specification of the workflow that's being implemented, both to make it clear what's actually going on under the hood, but also as a document to sanity check actual behavior of the Python code against.

This document also attempts to provide some reasonings/justification behind why the Git workflow is implemented that way. The modes are listed in the order that your team is likely to use them in practice.

In these examples, we'll use the following for stand-ins:

* libunicorn: the name of the third party project
* acme: our organization's "vendor name"

## import Mode

This mode imports a project from somewhere (currently, an archive file or a URL); projects are only intended to be imported once. (See Update Mode for updating the source.)

Import mode effectively performs the equivalent of:

| Command  | Description  |
| ---------------------- | ------------- |
| `git checkout -b libunicorn-upstream`  | <details>Create the branch that will contain the upstream source code.</details> |
| `git rm *` | <details>Clear that branch out. The code actually is more thorough than this,  including hidden files and directories.</details> |
| `unzip`/`tar xfvz` | <details>Unpack the provided archive of source code into the (now empty) working directory.</details> |
| `git add .` | <details>Add the source code that was just unarchived to the Git repository.</details>|
| Validate the files were added correctly | <details>A surprising number of projects ship files in their source archives that have been `.gitignored`; this code validates that all files in the tarball were added, and _warns_ the user if that is not the case.</details> |
| `git commit -m "Import libunicorn, version 1.0.0"` | <details>Commit the source code, using a standard message with details of where the source came from.</details> |
| `git tag libunicorn-1.0.0` | <details>Create a tag to easily identify this specific version of the upstream, pristine source code.</details> |
| `git push --tags origin libunicorn-upstream` | <details>Publish the branch and tag to everyone else. **NOTE**: `git-vendor-mirror` does _not_ automatically do this step; it informs the user that they need to do this step themselves, and provides the correct commands to type.</details> |

## resume-import Mode

As described above, some tarballs contain files that should be added, but aren't added as expected.

In this case, `git-vendor-mirror` halts the import process, and informs the user of how to resolve the problem, _should they want to include these files_. (In most cases, we recommend that you **do** include these files, using `git add -f`; as metnioned, if you get into this situation, `git-vendor-mirror` will help you out!)

After that's done, you need to finish up the last part of the `import` process; that it was the resume-import mode is for. In effect, it runs about the last half of the `import` process:

| Command  | Description  |
| ---------------------- | ------------- |
| `git tag libunicorn-1.0.0` | <details>Create a tag to easily identify this specific version of the upstream, pristine source code.</details> |
| `git push --tags origin libunicorn-upstream` | **NOTE**: `git-vendor-mirror` does _not_ automatically do this step. <details>Publish the branch and tag to everyone else. **NOTE**: `git-vendor-mirror` does _not_ automatically do this step; it informs the user that they need to do this step themselves, and provides the correct commands to type.</details> |


## setup-vendor-tracking-branch Mode

This mode sets up the vendor tracking branch. This branch creates a place where internal users can commit local patches on top of the imported source code, for their own purposes. This branch also serves as a great default branch for people in the organization to use, since any commits they make will automatically be in the "correct" place in the `git-vendor-mirror` world.

Users are informed they should invoke this mode immediately after the `import` mode has completed. 

| Command  | Description  |
| ---------------------- | ------------- |
| `git checkout libunicorn-upstream` | <details>Check out the just-imported copy of the source code into the working directory.</details> |
| `git checkout -b acme-master` | <details>Create the vendor branch to store local patches on.</details> |
| `git push origin acme-master` | <details>Publish the vendor branch that was just created for others to use.</details> |

## Update Mode

`update` mode is used to import a _newer version_ of the source code, `libunicorn 2.0.0`, for instance.

In many ways, it is similar to `import` mode (in fact, some of the code is shared for those two operations), but it does have some differences.

| Command  | Description  |
| ---------------------- | ------------- |
| `git checkout libunicorn-upstream`  | <details>Check out the branch where the pristine upstream source code is, in order to update it.</details> |
| `git checkout -b libunicorn-v2.0.0-update` | <details>Create a temporary branch to perform the update on.</details> |
| `git rm *` | <details>Clear the update branch out. The code actually is more thorough than this, including hidden files and directories.</details> |
| `git commit -a` | <details>Commit the emptied-out branch, in preparation to add the new version of the source code.</details> |
| `unzip`/`tar xfvz` | <details>Unpack the provided archive of source code into the (now empty) working directory.</details> |
| `git add .` | <details>Add the source code that was just unarchived to the Git repository. </details> |
| Validate the files were added correctly | <details>A surprising number of projects ship files in their source archives that have been `.gitignored`; this code validates that all files in the tarball were added, and _warns_ the user if that is not the case.</details> |
| `git commit -m "Import libunicorn, version 2.0.0"` | <details>Commit the updated source code.</details> |
| `git checkout libunicorn-upstream` | <details>Checkout the upstream branch; at this point, it should contain the previous version of the pristine, upsream source code, i.e. libunicorn version 1.0.0</details> |
| `git merge --ff-only --squash libunicorn-v2.0.0-update` | <details>Squash all the commits (in effect, the cleaning of the old source code and the adding of the new source code) into a single commit, and fast-forward merge that back to the `libunicorn-upstream` branch. Using Git to squash the removal and addition of the source code is an easy way to ensure that process is done correctly (using Git's internal logic for this); and merging as a fast-foward merge ensures we don't create a (pointlessly noisy) merge commit on the `libunicorn-upstream` branch.</details> |
| `git commit -a` | <details>Commit the squashed/merged branch to the `libunicorn-upstream` branch.</details> |
| `git tag libunicorn-2.0.0` | <details>Create a tag to easily identify this specific version of the upstream, pristine source code.</details> |
| `git branch -D libunicorn-v2.0.0-update` | <details>Delete the (now merged, and therefore useless) update branch.</details> |
| `git push --tags origin libunicorn-upstream` | **NOTE**: `git-vendor-mirror` does _not_ automatically do this step. <details>Publish the branch and tag to everyone else.</details> **NOTE**: `git-vendor-mirror` does _not_ automatically do this step; it informs the user that they need to do this step themselves, and provides the correct commands to type. |

Note that this mode can fail in the same way an `import` can fail, namely because missing added files are found or the user wants to otherwise modify/`--amend` the Git commit (as you can in `import` mode). In this case, the update process is left in a "halfway completed" state and the user is prompted to use the `resume-update` mode to complete the update.

## resume-update mode

This mode can be necessary if the user needs to modify the state of the commit containing the new, updated source code. It allows the user to complete the rest of the update process. It effectively runs the last few commands of the `update` process:

| Command  | Description  |
| ---------------------- | ------------- |
| `git merge --ff-only --squash libunicorn-v2.0.0-update` | <details>Squash all the commits (in effect, the cleaning of the old source code and the adding of the new source code) into a single commit, and fast-forward merge that back to the `libunicorn-upstream` branch. Using Git to squash the removal and addition of the source code is an easy way to ensure that process is done correctly (using Git's internal logic for this); and merging as a fast-foward merge ensures we don't create a (pointlessly noisy) merge commit on the `libunicorn-upstream` branch </details>|
| `git commit -a` | <details>Commit the squashed/merged branch to the `libunicorn-upstream` branch.</details> |
| `git tag libunicorn-2.0.0` | <details>Create a tag to easily identify this specific version of the upstream, pristine source code.</details> |
| `git branch -D libunicorn-v2.0.0-update` | <details>Delete (now merged, and therefore useless) update branch.</details> |
| `git push --tags origin libunicorn-upstream` | **NOTE**: `git-vendor-mirror` does _not_ automatically do this step. <details>Publish the branch and tag to everyone else. **NOTE**: `git-vendor-mirror` does _not_ automatically do this step; it informs the user that they need to do this step themselves, and provides the correct commands to type.</details> |

