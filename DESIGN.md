
# Design Intro

We often describe `git-vendor-mirror` as "bringing the CVS vendor branch concept to the Git(Hub) world."

Many developers often reply with "Why in God's name would I want to do that?"

This document attempts to explain the reasonings behind why not-relying on Git(hub) forks (even private ones) is a good (or, at least, useful) idea. 

That said, the key point to make up front: `git-vendor-mirrror` is not intended to (solely) facilitate developer productivity or developer needs. It takes into account other people (and their requirements) in the software delivery chain.

It's largely intended to serve a community of people with different (_and yet still important_) requirements that "just using GitHub forks" does not serve or address. These people include release engineers, QA, managers, and legal teams. It may even include other developers who don't want a project's life story, and just want to use a third party project, but be able to maintain it effectively and in a standardized way, that is incredibly clear to other developers.

If none of those perspectives or requirements are compelling to you, then we concede that it may be difficult to understand why a tool like `git-vendor-mirror` exists.

# Design Considerations

- **Isolation**: We consider isolation from the day-to-day machinations of an open source project to be a feature, not a bug. Historically, people would go to an FTP or other distribution site to download a tarball representing an official release of a third-party project, and use that in their software development. When "social coding" came along, it became fashionable to "just fork" projects, which removes _all_ buffers between day-to-day goings on of the project and some semblance of stability and "formalized releases."
  - **Isolation from instability**: for various reason, many open source projects don't have the resources to fully test their product for every commit. This may be "poor practice," but it's also an "industrial reality." A "formal release" (with version bump) of the software is the way a group of people signal to their consumers that they've provided some level of additional vetting, and may have performed additional tasks that are useful to consumers (running the various GNU autotools chain to generate `./configure`, etc.) `git-vendor-mirror` encourages you to pay attention to these (sometimes subtle) signals, and used the released versions of third-party open source projects, not "whatever the version happened to be when I clicked the GitHub fork button." (That said, `git-vendor-mirror` supports taking those drops, _should you want_ a random version of an upstream project.)
  - **Isolation from project dynamics**: Every open source project does things slightly differently. Which style of "(GitHub) pull request merging" is used, how (or even _whether_) they version code (with a tag), whether (and who) can force-push to the repo (potentially destroying work you rely on) are just a few examples. Keeping track of these differences is hard and annoying, especially if you don't personally agree with the "Git(Hub) aesthetics" they've chosen. `git-vendor-mirror` removes this layer of information by reverting back to "source drops", _while still facilitating developers who **want** to be involved in the community._ But it doesn't require you familiarize yourself with all these details just to use the source code from a project.


- **Reproducibility**: part of [owning your own software supply chain](http://whoownsmysoftwaresupplychain.com) is being able to produce all of the components used within a build process so as to be able to reproduce a build provided to a customer. Unfortunately, the Wild Wild West of GitHub forking (or pointing submodules to other, external projects) has created a situation where the commits and code you pointed to can suddenly disappear. `git-vendor-mirror` ensures that once you grab source code and import it, it will be available, _always_. 

- **Simplicity**: You might say "Oh, we've solved that problem! We use submodules in our build, but point them to private forks of code we maintain." The issue here is that all-too-often, updating that private fork falls to the person with the most time available. This may or may not be the person who originally forked the code. So they may not know the proper Git commands to pull in the new code from the upstream fork (if there even is such a thing as a "proper" Git command for that workflow, since Git, inexplicably, _prides_ itself on supporting every workflow under the sun). They may not know the proper upstream branch to pull to get the new release. They may not know how to get custom-patches onto that new branch. Or they may know all those things, but because they're not the same person who originally pulled the code in, they decide to use a different Git workflow, complicating (or even destroying) the history. This may seem like a hypothetical, but it's a problem we've seen teams run into all-too-often. `git-vendor-mirror` establishes _and automates_ a workflow that is easy for any developer, release or QA engineer, operations person, manager, or even executive to examine the repository and quickly, easily, and clearly discern _what third-party code is actually shipping in the product_.


- **Workflow Standardization**: to the above point of simplicity, `git-vendor-mirror` is fairly opinionated about workflow... as are most Git users. We take the simplest workflow approach that keeps things clear. We value human factors and human comprehendability and discoverability over whatever new whiz-bang feature the Git developers are trying to sell.

- **The Entire World Isn't Always GitHub Every-Time-All-the-Time**: GitHub is great. Lots of people and projects use it and love it. Others can't use it. This makes the "just use a GitHub fork"-solution unavailable to those folks. `git-vendor-mirror` doesn't care where you get your source code, so it's great for managing source code for situations where you need GitHub-hosted code, but don't host the code that needs that code on GitHub.

# Design Non-goals

The following are design goals and use cases which `git-vendor-mirror` specifically considered, and either does not optimize _or_ does not purport to solve the problem and/or use case.

- **Project participation**: `git-vendor-mirror` is not primarily designed with facilitating _participation_ in an open source project that you may also be using to ship a product to customers. _That said_, `git-vendor-mirror` _does_ support a workflow wherein you can create pull requests from your private, customized patches to share them with the community. (This is why we try to avoid using standardized branch names, like `master`.) But this is on a per-developer basis, and not a workflow that we're particularly concerned with facilitating. (We do, however, consider it important that we make patch-sharing _possible_, because we think sharing code back with the community is what makes open source work, and is important.)

- **Bike-shedding**: `git-vendor-mirror` is fairly (and unabashedly) opinionated about version control concepts and, specifically, Git workflows. Many people are, so that's not particularly noteworthy. Our opinions are framed from a release engineering point-of-view, and from over a decade of experience of having to support not only (various types of) developer use-cases, but _other people's use cases as well_. So while our opinions are strongly held, they may seem weird if you've never been a release engineer or a QA engineer or an operations person. But they come from years of listening to those people's problems and trying to solve them in a way that helps _both_ developers and not-developers.

- **Continuous third-party source consumption** (or You Don't Actually Want Continuous Delivery): You may actually want Continuous Delivery, but of your product... _not_ of the third-part components that you rely on to build your product. You probably want known drops of code with _some known level of quality_ (even if it's not high) in the case of third-party code. `git-vendor-mirror` optimizes for that, which means it specifically isn't designed to help with keeping every third-party library you use at bleeding edge.

See [WORKFLOW.md](https://github.com/preed/git-vendor-mirror/blob/master/WORKFLOW.md) for additional information.
