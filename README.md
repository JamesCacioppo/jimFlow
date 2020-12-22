# The jimFlow Branching Strategy
## There can be only one...
The chosen branching strategy is jimFlow.  It is a strategy which promotes simplicity and flexibility while striving to avoid "merge hell."  It's basically Github flow.

# Why are we using this workflow?
The Trunk branching strategy forces all developers to work in the trunk which is great in ideal conditions but can cause issues in non-ideal environments.  Gitflow, while solving many issues also adds much complexity and introduces an experience commonly known as "merge hell."

Github flow is a simple, flexible, and highly effective branching strategy.  What follows is our specification of that strategy.

Some goals of this strategy are:
* Prevent difficult merge conflicts which come from attempting to shoehorn hotfixes and release branches into both Master and Develop branches simultaneously.
* Avoid complexity and clutter by reducing the number of long running branches.
* Reduce branch lifetime by reducing time to merge.

Caveats:
* Going back and applying a fix to an older release version might require creating a branch that is hard or impossible to merge back into `master`.  See [Applying fixes to old releases](#applying-fixes-to-old-releases).

The different types of branches we may use are:
* Master
* Feature, bugfix, and hotfix branches

# Branches
![jimFlow diagram](jimFlow.png)
## Master
The Master branch will be named `master` and will contain the entire history.  All branches will merge back into master eventually.  Releases will be tagged commits.  CI pipelines will monitor `master` for tags with symver version in the format `v#.#.#` and build on these.

## Feature, Bugfix, and Hotfix
* Because all branches have effectively the same behavior they become much more flexible and the differences between them become more of a naming convention than anything else.
* Feature branches are used when developing a new feature or enhancement.  They are flexible as there may be cases where it makes sense to branch a feature off of a feature allowing multiple developers to work on related code.  In this event the branch should be merged back into the source if possible.  If the source branch has been merged into `master` then the developer will merge his/her branch into `master` as well.
* Feature branches should correspond with a story in Jira.  Long running branches should be avoided.  If multiple feature branches are required for a feature work should be done and merged such that the first branches don't depend on later branches.  This is so that merging one branch of a feature doesn't break the build.  Often this just means hooking up the UI portion last.
* During the lifetime of the feature branch the developer should keep an eye on the source branch and should frequently merge the source branch back into the feature branch.  This is done to avoid large merge conflicts.  Time to handle merge conflicts should be accounted for.
* Feature branch names should have a prefix of the Jira story number followed by a brief and human readable description of the work contained within the branch or the name of the story itself (when using Jira to create branches).  The fields should be separated by hyphens and slashes will not be used.  The syntax is `[story number]-[brief-description]`.
    * Example: `SSP1-246-add-leveldb`.  In this case `SSP1-246` was the story in Jira and `add-leveldb` describes the content which was the addition of LevelDB to the container image.
    * Allowing the use of short descriptions instead of the ticket name prevents branches with duplicate names in the event it makes sense to have multiple branches for the same story.
* Bugfix and Hotfix branches should follow the same proceedures.
* Bugfixes may branch from both `master` and feature branches.
* Hotfixes will branch from `master` except in cases as described in [Applying fixes to old releases](#applying-fixes-to-old-releases).

# Examples
## Feature branches
* Creating a feature branch
```
git checkout master
git checkout -b STORY3-add-node-feature
git push origin STORY3-add-node-feature
```

* Merging the source branch into the feature branch before a PR
```
git checkout STORY3-add-node-feature
git merge master
```
## The `master` branch
* Tagging releases
```
git checkout master
git tag -a v1.0.1 -m "some meaningful message"
git push origin v1.0.1
```

## Applying fixes to old releases
It's best to avoid going back and fixing old releases but if you must then it might be a good idea to follow the trunk workflow in which you would create a branch from the release and commit to that branch.
```
git checkout v0.1.0
git checkout -b HOTFIX2-some-fix-ticket-name
```
When the fix is pushed tag the commit as shown in [The master branch](#the-master-branch) (obviously, skip the checkout master part).  Once the fix has been deployed and you're comfortable that everything works, go ahead and delete the branch.
```
git push --delete origin HOTFIX2-some-fix-ticket-name
git fetch && git remote prune origin
```
If there's ever a need to re-open that branch and make further changes just find the SHA1 for the last commit on the deleted branch with `git reflog --no-abbrev` and then check it out with `git checkout -b <branch> <sha>`.