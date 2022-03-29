Git Best Practices
============================

Commit early and often
-------------------------

Git only takes full responsibility for your data when you **commit**. If you fail to commit and then do something poorly thought out, you can run
into trouble.
 *Commit Early And Often*. 
 If, after you are done, you want to pretend to theoutside world that your work sprung complete from your mind into the repository in utter perfection with each concept fully thought out and
 divided into individual concept-commits,

Don't panic
-----------

As long as you have committed your work (or in many cases even added it
with `git add`) your work will not be lost for at least two weeks unless you really work at it (run commands that manually purge it).

When attempting to find your lost commits, first make *sure* you will
not lose any current work. You should commit or stash your current work
before performing any recovery efforts that might destroy your current
work and perhaps take backups of it (see [Backups](#backups) below).
After finding the commits you can `reset`, `rebase`, `cherry-pick`,
`merge`, or otherwise do what is
necessary to get the commit history and work tree you desire.

There are three places where "lost" changes can be hiding. They might be
in the reflog (`git log -g`), they might be in lost&found
(`git fsck --unreachable`), or they might have been stashed (`git stash list`).

-   reflog

    The reflog is where you should look first and by default. It shows
    you each commit that modified the git repository. You can use it to
    find the commit name (SHA-1) of the state of the repository before
    (and after) you typed that command. While you are free to go through
    the reflog manually (`git log -g`
    ) or searching it (`git log -Sfoo -g`), you can also visualize the 
    repository using the following command (Look
    for dots without children and without green labels):

    ```bash
    gitk --all --date-order $(git log -g --pretty=%H)
    ```

-   Lost and found

    Commits or other git data that are no longer reachable through any
    reference name (branch, tag, etc) are called "dangling" and may be
    found using `fsck`. There
    are legitimate reasons why objects may be dangling through standard
    actions and normally over 99% of them are entirely uninteresting for
    this reason.

    -   Dangling Commit

        These are the most likely candidates for finding lost data. A
        dangling commit is a commit no longer reachable by any branch or
        tag. This can happen due to resets and rebases and are normal.
        `git show SHA-1` will
        let you inspect them.

        The following command helps you visualize these dangling
        commits. Look for dots without children and without green
        labels.

        ```bash
        gitk --all --date-order $(git fsck | grep "dangling commit" | awk '{print $3;}')
        ```

    -   Dangling Blob

        A dangling blob is a file that was not attached to a commit.
        This is often caused by `git add`s that were superceded before commit 
        or mergeconflicts. Inspect these files with

        ```bash
        git show SHA-1
        ```

    -   Dangling Tree

        A dangling tree is a directory tree of files that was not
        attached to a commit. These are rarely interesting, and often
        caused by merge conflicts. Inspect these files with
        `git ls-tree -r SHA-1`

-   Stashes

    Finally, you may have stashed the data instead of committing it and
    then forgotten about it. You can use the
    `git stash list` command or
    inspect them visually using:

    ```bash
    gitk --all --date-order $(git stash list | awk -F: '{print $1};')
    ```

-   Misplaced

    Another option is that your commit is not lost. Perhaps the commit
    was just made on a different branch from what you remember. Using
    `git log -Sfoo --all` and `gitk --all --date-order` to
    try and hunt for your commits on known branches.

-   Look elsewhere

    Finally, you should check your backups, testing copies, ask the
    other people who have a copy of the repo, and look in other repos.


Don't change published history
------------------------------

Once you `git push` (or in theory someone pulls from your repo, 
but people who pull from a workingrepo often deserve what they get) 
your changes to the authoritative
upstream repository or otherwise make the commits or tags publicly
visible, you should ideally consider those commits etched in diamond for
all eternity. If you later find out that you messed up, make new commits
that fix the problems (possibly by revert, possibly by patching, etc).


### Branch workflows

Answering the following questions helps you choose a branch workflow:

-   Where do important phases of development occur?
-   How can you identify (and backport) groups of related change?
-   Do you have work which often needs to be updated in multiple
    distinct long-lived branches?
-   What happens when emergency patches are required?
-   What should a branch for a particular purpose (including
    user-tactical) be named?
-   What is the lifecycle of a branch?

See the following references for more information on branch workflows.




Branch naming conventions are also often overlooked. You must have
conventions for naming release branches, integration branches, QA
branches, feature branches (if applicable), tactical branches, team
branches, user branches, etc. Also, if you use share repositories with
other projects/groups, you probably will need a way to disambiguate your
branches from their branches. Don't be afraid of "/" in the branch name
when appropriate (but do be afraid of using a remote's name as a
directory component of a branch name, or correspondingly naming a remote
after a branch name or directory component).



### Release tagging

Choosing your release workflow (how to get the code to the customer) is
another important decision. You should have already considered most of
the issues when going over the branching and distributed workflow above,
but less obviously, it may affect how and when you perform tagging, and
specifically the name of the tag you use.



Two ideas for your consideration. Instead of a release tag, use a
release branch with the marketing name (and then stop committing to that
branch after release, disabling write access to it 

### Security model

For instance, did you decide certain branches should only have certain
people being allowed to access it? Did you decide that certain
repositories should only have certain people able to access/write to
them?

While git allows users to set up many different types of access control,
access methods, and the like; the best for most deployments might be to
set up a centralized git master repository with a
[gitolite](https://github.com/sitaramc/gitolite) manager to provide fine
grained access control with ssh based authentication and encryption.

Of course, security is more than access control. It is also assurance
that what you release is what was written by the people it should be
written by, and what was tested. Git provides you this for free, but
certain formal users may wish to use signed tags. Watch for signed
pushes in a future version of git.

Do divide work into repositories
--------------------------------

Repositories sometimes get used to store things that they should not,
simply because they were there. Try to avoid doing so.

-   One conceptual group per repository.

    Does this mean one per product, program, library, class? Only you
    can say. However, dividing stuff up later is annoying and leads to
    rewriting public history or duplicative or missing history. Dividing
    it up correctly beforehand is much better.

-   Read access control is at the repo level

    If someone has access to a repository, they have access to the
    entire repo, all branches, all history, everything. If you need to
    compartmentalize read access, separate the compartments into
    different repositories.

-   Separate repositories for files that might be needed by multiple
    projects

    This promotes sharing and code reuse, and is highly recommended.

-   Separate repositories for large binary files

    Git doesn't handle large binary files ideally yet and large
    repositories can be slow. If you must commit them, separating them
    out into their own repository can make things more efficient.

-   Separate repositories for planned continual history rewrites

    You will note that I have already recommended against rewriting
    public history. Well, there are times when doing that just makes
    sense. One example might be a cache of pre-built binaries so that
    most people don't need to rebuild them. Yet older versions of this
    cache (or at least older versions not at tag boundaries) may be
    entirely useless and so you want to pretend they never happened to
    save space. You can rebase, filter, or squash these unwanted commits
    away, but this is rewriting history and can cause problem. So if you
    really must do so, isolate these files into a repository so that at
    least everything else will not be affected.

-   Group concepts into a superproject

    Once you have divided, now you need to conquer. You can assemble
    multiple individual repositories into a superproject to group all of
    the concepts together to create your unified work.

    There are two main methods of doing this.

    -   `git-submodules`

        Git submodules is the native git approach, which provides a
        strong binding between the superproject repository and the
        subproject repositories for every commit. This leads to a
        baroque and annoying process for updating the subproject.
        However, if you do not control the subproject (solvable by
        "forking") or like to perform blame-based history archeology
        where you want to find out the absolute correspondence between
        the different projects at every commit, it is very useful.

    -   `gitslave`

        [gitslave](https://gitslave.sf.net) is a useful tool to add a
        subsidiary git repositories to a git superproject when you
        control and develop on the subprojects at more or less the same
        time as the superproject, and furthermore when you typically
        want to tag, branch, push, pull, etc all repositories at the
        same time. There is no strict correspondence between
        superproject and subproject repositories except at tag
        boundaries (though if you need to look back into history you can
        usually guess pretty well and in any case this is rarely
        needed).

Do make useful commit messages
------------------------------

Creating insightful and descriptive commit messages is one of the best
things you can do for others who use the repository. It lets people
quickly understand changes without having to read code. When doing
history archeology to answer some question, good commit messages
likewise become very important.

The normal git rule of using the first line to provide a short (50-72
character) summary of the change is also very good. Looking at the
output of `gitk` or
`git log --oneline` might help
you understand why.

Also see [A Note About Git Commit
Messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
for even more good ideas.

While this relates to the later topic of [integration with external
tools](#integration), including bug/issue/request tracking numbers in
your commit messages provides a great deal of associated information to
people trying to understand what is going on. You should also enforce
your standards on commit messages, when possible, through hooks. See
[Enforcing standards](#enforce) below.

On Sausage Making
-----------------

Some people like to hide the sausage making[¹](#sausage_metaphor), or in
other words pretend to the outside world that their commits sprung
full-formed in utter perfection into their git repository. Certain large
public projects demand this, others demand smushing all work into one
large commit, and still others do not care.

A good reason to hide the sausage making is if you feel you may be
cherry-picking commits a lot (though this too is often a sign of bad
workflow). Having one or a small number of commits to pick is much
easier than having to find one commit here, one there, and half of this
other one. The latter approach makes your problem much much harder and
typically will lead to merge conflicts when the donor branch is finally
merged in.

Another good reason is to ensure each commit compiles and/or passes
regression tests, and represents a different easily understood concepts.
The former allows `git-bisect` to choose any commit and have a good chance of that commit doing 
something useful, and the latter allows for easy change/commit/code
review, understanding, archeology, and cherry-picking. When reviewing
commits, for example the reviewer might see something suspicious in a
commit and then have to spend time tracking down their suspicions and
write them up, only to discover five commits later that the original
developer subsequently found and fixed the problem, wasting the
reviewer's time (reviewing the entire patch series as a diff fixes this
problem but greatly adds complexity as multiple concepts get muddled).
By cleaning up patches into single, logical changes that build on one
another, and which don't individually regress (i.e., they are always
moving towards some desirable common endpoint), the author is writing a
chronological story not of what happened, but what *should* happen, with
the intent that the audience (i.e., reviewers) are convinced that the
change is the right thing to do. Proponents claim it is all about
leaving a history others can later use to understand *why* the code
became the way it is now, to make it less likely for others to break it.

The downside to *hiding the sausage* making is the added time it takes
to perfect the administrative parts of the developers job. It is time
taken away from getting code working; time solely dedicated to either
administrative beauty or enhancing the ability to perform the
blame-based (or ego-full) development methodology.

If you think about it, movies are made this way. Scenes are shot out of
temporal order, multiple times, and different bits are picked from this
camera and that camera. Without examining the analogy too closely, this
is similar to how different git commits might be viewed. Once you have
everything in the "can" (repository) you go back and in post-production,
you edit and splice everything together to form individual cuts and
scenes, sometimes perhaps even doing some digital editing of the
resulting product.

`git rebase -i`, `git add -p`, and `git reset -p` can fix commits
up in post-production by splitting different concepts, merging fixes to
older commits, etc. See [Post-Production Editing using
Git](https://sethrobertson.github.io/GitPostProduction/) also
[TopGit](https://repo.or.cz/w/topgit.git) and
[StGit](https://stacked-git.github.io/).

Be sure you do all of this work *before* doing any non-squashed merges
(not rebases: merges) and *before* pushing. Your work becomes much more
complex and/or impossible afterwards.

¹ The process of developing software, similar to the process of making
sausage, is a messy messy
business[²](#never_discover_how_sausages_are_made); all sorts of stuff
happens in the process of developing software. Bugs are inserted into
the code, uncovered, patched over. The end result may be a tasty
program, but anyone looking at the process of how it was created
(through inspection of the commits) may end up with a sour taste in
their mouth. If you hide the sausage making, you can create a beautiful
looking history where each step looks as delicious as the end-product.
[Back to footnote reference.](#sausage)

² If you do not understand why someone would want to hide the sausage
making, and you enjoy eating sausage, never, ever, watch sausages being
made, read ["The Jungle"](https://en.wikipedia.org/wiki/The_Jungle), or
otherwise try to expose yourself to any part of the sausage making
process. You will lead a much tastier (and perhaps shorter) life in your
blissful ignorance.

Keep up to date
------------------

This section has some overlap with workflow. Exactly how and when you
update your branches and repositories is very much associated with the
desired workflow. Also I will note that not everyone agrees with these
ideas (but they should!)

-   Pulling with `--rebase`

    Whenever I pull, under most circumstances I
    `git pull --rebase`. This is
    because I like to see a linear history (my commit came after all
    commits that were pushed before it, instead of being developed in
    parallel). It makes history visualization much simpler and
    `git   bisect` easier to see
    and understand.

    A specific circumstance in which you should avoid using
    `git pull   --rebase` is if
    you merged since your last push. You might want to
    `git fetch; git rebase -p @{u}`
    (and check to make sure the merge was recreated
    properly) or do a normal merge in that circumstance.

    Another specific circumstance is if you are pulling from a
    non-authoritative repository which is not fully up to date with
    respect to your authoritative upstream. A rebase in this
    circumstance could cause the published history to be rewritten,
    [which would be bad](#pubonce).

    Some people argue against this because the non-final commits may
    lose whatever testing those non-final commits might have had since
    the deltas would be applied to a new base. This in turn might make
    git-bisect's job harder since some commits might refer to broken
    trees, but really this is only relevant to people who want to hide
    the sausage making. Of course to *really* hide the sausage making
    you should still rebase and then test each intermediate commit to
    ensure it compiles and passes your regression tests (you do have
    regression tests, don't you?) so that a future bisector will have
    some strong hope that the commit will be usable. After all, that
    future bisector might be you.

    Other people argue against this (especially in highly decentralized
    environments) because doing a merge explicitly records who performed
    the merge, which provides someone to blame for inadequate testing if
    two histories were not combined properly (as opposed to the hidden
    history with implicit blame of rebase).

    Still others argue that you are unable to automatically discover
    when someone else has [rewritten public history](#pubonce) if you
    use `git   pull --rebase`
    normally, so someone might have hidden something malicious in an
    older (presumably already reviewed) commit. If this is of concern,
    you can still use rebase, but you would have to
    `git   fetch` first and look
    for "forced update" in that output or in the reflog for the remote
    branches.

    You can make this the default with the "branch.\<name\>.rebase"
    configuration option (and more practically, by the
    "branch.autosetuprebase" configuration option). See [man
    git-config](https://gitirc.eu/git-config.html).

-   Rebasing (when possible)

    Whenever I have a private branch that I want to update, I use rebase
    (for the same reasons as above). History is clean and simple.
    However, if you share this branch with other people, rebasing is
    rewriting public history and should/must be avoided. You may only
    rebase commits that no-one else has seen (which is why
    `git pull   --rebase` is
    safe).

-   Merging without speeding

    `git merge` has the concept
    of fast-forwarding, or realizing that the code you are trying to
    merge in is identical to the result of the code after the merge.
    Thus instead of doing work, creating new commits, etc, git simply
    changes the branch pointers (fast forwards them) and calls it good.

    This is good when doing `git pull`
    but not so good when doing
    `git   merge` with a
    non-@{u} (upstream) branch. The reason this is not good is because
    it loses information. Specifically it loses track of which branch is
    the first parent and which is not. If you don't ever want to look
    back into history, then it does not matter. However, if you want to
    know the branch on which a commit was originally made, using
    fast-forward makes that question impossible to answer. If you try,
    git will pick one branch or the other (the first parent or second
    parent) as the one on which both branches' activities were performed
    and the other (original) parent's branch will be anonymous. There
    are typically worse things in the world, but you lose information
    that is not recoverable in any other way by a repository observer
    and in my book that is bad. Use
    `git merge   --no-ff`
    instead.

Do periodic maintenance
-----------------------

The first two items should be run on your server repositories as well as
your user repositories.

-   Validate your repo is sane (`git fsck`)

    You need not check dangling objects unless you are missing something

-   Compact your repo (`git gc` and `git gc --aggressive`)

    This will removed outdated dangling objects (after the two+ week
    grace period). It will also compress any loose objects git has added
    since your last gc. git will run a minimal gc automatically after
    certain commands, but doing a manual gc often (and "–aggressive"
    every few hundred changesets) will save space and speed git
    operations.

-   Prune your remote tracking branches
    (`git remote update --prune`
    )

    This will get rid of any branches that were deleted upstream since
    you cloned/pruned. It normally isn't a major problem one way or
    another, but it might lead to confusion.

-   Check your stash for forgotten work
    (`git stash list`)

    If you don't do it very often, the context for the stashed work will
    be forgotten when you finally do stumble on it, creating confusion.

Follow common coding standards
--------------------

Having standards is a best practice and will improve the quality of commits, code-base, its readability, maintainability, extensibility.



See [Enforcing Coding Standards in
Husky](https://www.wisdomgeek.com/development/web-development/javascript/enforcing-coding-standards-husky-pre-commit-hooks/)
for an example for a "Git Update Hook" and "Git Pre-Commit Hook" that
enforces certain standards. Note that the update hook is examining files
individually instead of providing whole-repository testing. Whether
individual files can be tested in isolation for your standards or
whether you need the whole repository (for instance, any language where
one file can reference or include another might need whole repository
checks) is of course a personal choice. The referenced examples are
useful for ideas, anyway.



Miscellaneous "Do"s
-------------------

These are random best practices that are too minor or disconnected to go
in any other section.

-   Copy/move a file in a different commit from any changes to it

    If you care about git properly displaying the fact that you moved a
    file, you should copy or move the file in a different commit from
    any changes you need to immediately make to that file. This is
    because git does not record `git mv`
    any different from a delete and an add, and because `git cp` doesn't
    even exist. Git's output commands are the ones that interpret the
    data as a move or copy. See the -C and -M options to
    `git log` (and similar commands).

-   Name your stashes

    If you don't provide a name when stashing, git generates one
    automatically based on the previous commit. While this tells you the
    branch where a stash was made, it gives you no idea what is in it.
    Unless you plan to pop a stash in the next few minutes, you should
    always give it a name with `git stash save XXX` rather than the shorter default `git stash`. This way you'll have some idea
    what a stash is about when you are looking at it months later.

-   Protect your bare/server repos against history rewriting

    If you initialize a bare git repository with `–shared` it will
    automatically get the git-config `receive.denyNonFastForward` set
    to true. You should ensure that this is set just in case you did
    something weird during initialization. Furthermore, you should also
    set `receive.denyDeletes` so that people who are trying to rewrite
    history cannot simply delete the branch and then recreate it. Best
    practice is for there to be a speedbump any time someone is trying
    to delete or rewrite history, since it is such a bad idea.

-   Experiment!

    When you have an idea or are not sure what something does, try it
    out. Ideally try it out in a clone or copy so that recovery is
    trivial. While you can normally completely recover from any git
    experiment involving data that has been fully committed, perhaps you
    have not committed yet or perhaps you are not sure whether something
    falls in the category of "trying hard" to destroy history.

Miscellaneous "don't"s
----------------------

In this list of things to *not* do, it is important to remember that
there are legitimate reasons to do all of these. However, you should not
attempt any of these things without understanding the potential negative
effects of each and why they might be in a best practices "Don't" list.

*DO NOT*

-   commit anything that can be regenerated from other things that were
    committed.

    Things that can be regenerated include binaries, object files, jars,
 

-   commit configuration files

    Specifically configuration files that might change from environment

-   commit large binary files (when possible)

    After running a `git gc` you
    should be able to find the largest objects by running:

    ```bash
    git verify-pack -v .git/objects/pack/pack-*.idx |
    grep blob | sort -k3nr | head |
    while read s x b x; do
      git rev-list --all --objects | grep $s |
      awk '{print "'"$b"'",$0;}';
    done
    ```

-   create very large repositories (when possible)

-   use `reset`
    (`--hard | --merge`) without
    committing/stashing

    This can often overwrite the working directory without hope of
    recourse.

-   use `checkout` in file mode

    This will overwrite some (or potentially all with
    `.`) of the working
    directory without hope of recourse.

-   use `git clean` without
    previously running with "`-n`" first

    This will delete untracked files without hope of recourse.

-   prune the reflog

    This is removing your safety belt.

-   expire "now"

    This is cutting your safety belt.

-   use `git repack -ad`

    Unreferenced objects in a newly redundant pack will get deleted
    which cuts your safety belt. Instead use
    `git gc` or at least
    `git repack -Ad`

-   use a branch argument to `git pull` or `git fetch`

    No doubt there is a good use case for, say,
    `git pull origin master` or
    whatever, but I have yet to understand it. What I *do* understand is
    that every time I have seen someone use it, it has ended in tears.

-   use git as a generic filesystem backup tool

-   rewrite public history

    See [section about this topic](#pubonce). It bears repeating,
    though.

-   change where a tag points

    This is another way to [rewrite public history](#pubonce).

-   use `git-filter-branch`

    Still another way to [rewrite public history](#pubonce).

    However, if you are going to use git-filter-branch, make sure you
    end your command with \` –tag-name-filter cat – –all\` unless you
    are really really sure you know what you are doing.

Sources
-----------------

-   [Pro Git](https://git-scm.com/book/)
-   [Git for Compute Scientists](https://eagain.net/articles/git-for-computer-scientists/)
-   [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
-   [The Git Parable](https://tom.preston-werner.com/2009/05/19/the-git-parable.html)
-   [Documentation](https://git-scm.com/documentation)
