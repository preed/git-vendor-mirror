
# Intro

`git-vendor-mirror` is a tool with strong, release engineering-informed opinions on managing third-party source code, your software supply chain. It integrates with the software you're building. See [DESIGN.md](https://github.com/preed/git-vendor-mirror/blob/master/DESIGN.md) for more details on its history, its purpose, its design goals, and its design non-goals. See [WORKFLOW.md]((https://github.com/preed/git-vendor-mirror/blob/master/WORKFLOW.md) for a description of what `git-vendor-mirror` is actually doing under the hood.

`git-vendor-mirror` basically provides a way to "fork" GitHub (and non-GitHub!) repositories without actually employing the Git-concept of a fork. In effect, it's a tool to manage leveraging Git and Git-concepts to replicate the behavior of CVS's "vendor branch" pattern.

While it may seem weird to "go back to CVS," even if conceptually, we've found the simplistic model CVS employed to fall into a usecase the vast majority of software development organizations need, whereas pure Git forks are too complicated and introduce an incredible amount of overhead for those organizations. Again, DESIGN.md has more details on our position and the argument for `git-vendor-mirror`.

# Getting Started

Importing source code using `git-vendor-mirror` is a fairly simple process, involving two main steps: importing and vendor branch setup.

You will see reference to the concept of a "vendor-name"; that's a small string that describes your organization. (Examples: "moz" if your organization were Mozilla.) Technically, there are no limits on the length of this string, but we recommend a string of 2-4 characters. (The string gets used in tag names and such, so shorter helps.)

## Importing source code

The first step to importing source code is to create an empty Git(Hub) repository; specific instructions for doing this are beyond the scope of this document.

The second decision to make is where you will get the source from; two options are available: a local file, or downloaded from a URL. We recommend downloading from a URL if available. Currently, the only archive file format handled is `.zip`. (This was because the initial main use-case was GitHub, which serves project archives up as `.zip` files.)

In the following example, we'll import a released version of [rapidjson](https://github.com/miloyip/rapidjson) into a private github repository we've created. We'll use the vendor string `acme`.

To perform the source code import:

1. Clone the (empty) repository:

```
[you@machine workdir]$ git clone github.com:acme-private-repos/acme-rapidjson
Cloning into 'acme-rapidjson'...
warning: You appear to have cloned an empty repository.
[you@machine workdir]$
```

2. Go into the repo and change any settings you made need:

```
cd acme-rapidjson
git config user.name "My Actual Name"
git config user.email me@email.address.i.want.example.com
```

3. Run the actual import

`git-vendor-mirror` takes a "mode" argument to tell it what to do, and an additional number of other arguments to specify options and other required information. 

To import, the vendor name (`--vendor`), package name (`--package-name`), and package version (`--package-version`) are required.

In our case, the vendor name is `acme` and the package name is `rapidjson`.

The package version we'll be using is 1.1.0. See the notes below on "Released versus Unreleased Software" for considerations about which version to use and what string to use here. In our case, [1.1.0 is a released (tagged)](https://github.com/miloyip/rapidjson/releases/tag/v1.1.0) version of `rapidjson`.

We also need to tell `git-vendor-mirror` where we'll be getting this source code from; two options are currently available. Only one can be specified at a time:

* the URL to use to download the package (`--package-url`), or
* a path to a local archive file to import (`--package-file`)

Finally, `git-vendor-mirror` provides an option to "strip" directories off of an archive, similar to the `patch (1)` utility. Many archive packages put the source code into a top-level directory, i.e. `rapidjson-1.1.0`; if you imported the package directly, you'd have one (mostly useless) directory at the top-level of the repository.

The `--strip-path` option allows you to strip an arbitrary number of paths off the archive if necessary. In _most_ cases, you'll need this option, and the value will be `1`, to strip off the top-level directory.

Putting all these together, the command to run the import would look like the following, which has been split over several lines to improve readability. (We also used the short-form of the above arguments; you may want to look at `git-vendor-mirror --help`):

```
[you@machine workdir]$ ~/path-to/git-vendor-mirror/git-vendor-mirror \
      import \
      -V acme \
      -n rapidjson \
      -v 1.1.0 \
      -u https://github.com/miloyip/rapidjson/archive/v1.1.0.zip \
      -p 1
```

4. Inspect the result and publish

When the import process described in step 3 is completed, `git-vendor-mirror` will prompt you to use `git log` or `gitk` to inspect the work it has done. You can correct any problems (such as adding to the commit message, etc.) using `git commit --amend`.

`git-vendor-mirror` creates a variety of branches and tags to facilitate source code investigations later (i.e. "What modifications did we make to this source code? What changed between version 1.1.0 and 1.1.1?")

As such, you'll need to push those tags to the repository of record, in addition to the import itself. Additionally, the branch that we've put the source code on will be named `PROJECT_NAME-upstream`, to indicate it's the branch tracking the upstream code. So, in our example, it would be `rapidjson-upstream`.

When you're ready, push the result back to your repository of record:

`git push --tags origin rapidjson-upstream`

5. Set up the vendor-tracking branch

`git-vendor-mirror` is designed to set up and help you maintain a branch that can be used for private, custom patches to the source code, similar to how CVS handled vendor branches.

This step uses the `setup-vendor-tracking-branch` mode to set up this branch for you. You will only need to do this after an `import`, and you should do it immediately after that import.

You will need the vendor name and package name from before.

Back in the source directory you just imported:

```
[you@machine workdir]$ ~/path-to/git-vendor-mirror/git-vendor-mirror \
      setup-vendor-tracking-branch \
      -V acme \
      -n rapidjson
```

The script will give you further instructions on what to do when it runs, but generally it just involves pushing the result to your repository of record:

```
git push origin acme-master
```

6. Reset the default branch (optional):

Since you want all internal/private patches/development to occur on the `acme-master` branch (*not* `master`), it is often a good idea to reset the default branch that Git checks out for users upon a `clone` to this branch. Instructions to accomplish this for all the various Git repository management tools is beyond the scope of this document. But on Github, you can go to the following repository URL and change the default branch:

https://github.com/packetstash/acme-private-repos/acme-rapidjson/settings/branches

If you're going to do this (and we heavily suggest that you do), it's best to do this early, before anyone has cloned the repository for use.

7. That's it! You should use/develop against/point submodules for other (internal) projects to the `acme-master` branch.
