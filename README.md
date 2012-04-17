# Git Workflows

## Main Purpose

1. Introduce git workflow
2. Show what git can do, but not in detail

## Basics

* The Three States, **Most Important Concept**
    * working directory (anything outside .git)
    * staging area (a.k.a index -- .git/index)
    * git repository (.git)
* References
    * ancestry reference
        * `HEAD^{,1,2}`
        * `HEAD~{,1,2,...}`
        * even more fancy -- `HEAD~3^2`
    * (@) reflog, can help with retrieving lost commits -- `HEAD@{<n>}`
    * (@) `<branch1>..<branch2>`
    * (@) `<branch1>...<branch2>`
* Branch
* Workflow

## Before Coding

* Alias
    * `git config --global alias.st status`
    * `echo '[alias] st = status' << ~/.gitconfig`
* Color
    * `git config --global color.status auto`
    * also check `color.{branch,diff,interactive} auto`
* Log
    * `git log --format=oneline --graph`
    * `git log --all`
    * `git log -p`
    * even better: `git help log<CR>/PRETTY FORMATS` or <http://progit.org/book/ch2-3.html>

---

## One-Person Workflow

### Principle

* Commit early, commit often.
* Always know where you are -- `status` and `log`

### Workflow

1. No need. Code, add, commit, push. Nothing special.
2. Try to make your commit atomic and reasonable.
3. Rewrite history if needed.
4. Advanced:
    1. Feature branch with merge --no-ff: good for rollback but bad for bisect and blame
    2. Parallel development using branch and merge.

### Recipe

* Add
    * add new file
    * add new empty folder -- `touch .gitkeeper`
    * add new change
* Diff
    * `git diff`
    * `git diff {--cache,<ref>}`
    * `git diff --check` for trailing whitespace **(optional, better to setup your editor)**
* Commit
    * `git commit -m`
    * `git commit -am`
    * `git commit --amend{, -C HEAD}`
    * commit guidelines <http://progit.org/book/ch5-2.html>
        * start with a single line that’s no more than about 50 characters and that describes the changeset concisely
        * followed by a blank line, followed by a more detailed explanation
        * explain your motivation for the change and contrast its implementation with previous behaviour
        * use commands like 'add something' instead of 'I added something'
* Revert
    * unstage -- `git reset HEAD <file>`
    * discard -- `git checkout -- <file>`
    * revert commit -- `git revert <hash>` or `git revert <hash1>..<hash2>`
* Show
    * `git show <hash>:<path_to_file>`
* Ignore Sample File
    * ./.gitignore vs ~/.gitignore
    * `git rm --cached <file>` or `git rm --cached -r <folder>`
* Remote
    * remote
    * `git push {, -f}`
    * `git pull`
* Advanced
    * branch
        * `git branch <new-branch>`
        * `git checkout -b <new-branch>`
        * remove local branch -- `git branch -d <branch>`
        * remove remote branch -- `git push origin :<branch>`
    * merge
        * resolve conflict
            1. if status is fine then to 5 else to 2
            2. search for `&lt;&lt;&lt;&lt;&lt;&lt;&lt;`
            3. (maybe) communication
            4. add (again), to 1
            5. commit
        * `git merge --no-commit`
        * `git merge --no-ff`

---

## Basic Team Workflow

### Principle

* Use branch
* Public branch(master) is **atomic**, **test-passed** and **deployable**
* **Never** develop on public branch
* Private branch == draft
    > "Great books aren’t written– they’re rewritten." Michael Crichton

### Workflow

1. To work on a new feature, create a descriptively named **feature branch** off of master.
2. Commit to that branch locally and regularly push your work to the same named branch on the server. *(Push to server is optional: up to you)*
3. When you finish that feature, rewrite commit history using reset/rebase/squash or amend to make every commit atomic and reasonable, remove all checkpoint commit and rebase onto master.
4. Alternative, use `git merge --squash` to generate only-one commit.
5. Fast-forward merge that feature branch from master branch and push to server.
6. Remove remote and local branch.
7. *For fixing bugs, you may want to simplify the process using **auto rebase**. (optional)*

### Why?

1. Using branch, parallel developing, good for **hotfix**.
2. Clean, linear commit history, no merge commit, no checkpoint commit.
3. Every commit is atomic and well documented with commit message, good for `bisect` and **code review**.
4. Every commit is deployable, good for **rollback**.

### Recipe

* Switch Branch
    * `git checkout <branch>`
* Add
    * `git add -i`
    * `git add -p`
* Diff
    * `git diff --staged`
    * `git diff <ref1>..<ref2>` or `git diff <ref1> <ref2>`
    * `git diff <ref1>...<ref2>`
    * (@) `git merge-base <branch1> <branch2>`
* Reset
    * `git reset {--soft,--hard} <ref>`
* **Rebase**
    * `git rebase {,-i}`
        * pick, edit, squash, reword, fixup
        * remove, reorder
        * split
    * `git rebase {--abord,--continue}`
    * **never** rebase public branch
    * (@) `git rebase --onto <newbase> <upstream> <branch>`
    * (@) `filter-branch`
        * remove binary files in history
            `git filter-branch --tree-filter 'rm -f path/to/file.bin' HEAD`
* Fetch
    * `pull` == `fetch` + `merge`
    * `fetch` + `rebase`
    * `git pull --rebase`
* Auto Rebase
    * `git config branch.<branch>.rebase true`
    * `git config --global branch.autosetuprebase always`
* **Stash, WIP**
    * `git stash {,save <name>,save --patch <name>}`
    * `git stash list`
    * `git stash {apply,apply --index,pop,clear,drop}`
    * (@) `git stash branch`
* Remove Untracked Files
    1. `rm ./*`
    2. `git checkout .`
* Submodule
    * (@) add, init, update
    * (@) alternative: subtree merging
* Blame
    * `git blame -L <from>,<to> path/to/file`
    * ignore whitespace -- `git blame -w`
* Binary Search
    1. `git bisect start`
    2. `git bisect {good/bad}` or `git bisect run <path/to/script>`
    3. `git bisect reset`
* Retrieve 'Lost' Commits
    * `git cherry-pick <reflog>`
* Shortlog
    * `git log --pretty=short {filter} | git shortlog`
* Garbage Collection
    * `git gc`

---

## Open Project Workflow

> Open project means large public projects, including all kinds of software engineering. I searched for article about this kind of workflow but didn't get a perfect answer. So below is just my personal opinion, may be unrealistic and need improvement for sure. I'm **not experienced in this workflow**.

### Principle

* Remote collaboration is possible.
* Support multipul release version.

### Workflow

* Puclic branch including develop, master(or current) and releases.
* Merge reviewed pull request into develop.
* Create new branch for each release off of develop and commit for release meta data.
* Merge into master and develop and tag on master.
* Hotfix commit in release branch and cherry-pick into develop (and other later release branch if necessary), fast forward merge into master and tag.

### Recipe

* Patch
    * `git format-patch <target-ref>`
* Apply
    * `git apply {,--check } <patch-file>`
    * `am` == `apply` + `commit`
* Cherry-Pick
    * `git cherry-pick <ref>`
* Tag
* Archive
    * check `git help archive` examples
* Shortlog
