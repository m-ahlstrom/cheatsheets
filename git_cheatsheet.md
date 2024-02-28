# GIT CHEATSHEET

1. GIT MANEGEMENT (ssh, multiple accounts, git config, rev-parse, cat-file)
2. GIT BASICS (clone, init, add, commit, stash)
3. GIT HISTORY (log, shortlog, blame, reflog, tag)
4. GIT BRANCHES (branch, checkout, diff, merge)
5. GIT REMOTE REPOSITORIES (remote, push, pull, fetch)
6. GIT UNDO CHANGES (reset, restore, revert, cherry-pick, rebase)
7. GIT OPTIMIZATION (submodules, subtrees, commit-graph)
8. GIT IN PRACTICE (examples from Oh Shit Git, Atlassian, etc.)

## USEFUL RESOURCES

- [Git’s database internals I: packed object store](https://github.blog/2022-08-29-gits-database-internals-i-packed-object-store/)
- [Git’s database internals II: commit history queries](https://github.blog/2022-08-30-gits-database-internals-ii-commit-history-queries/)
- [Git’s database internals III: file history queries](https://github.blog/2022-08-31-gits-database-internals-iii-file-history-queries/)
- [Git’s database internals IV: distributed synchronization](https://github.blog/2022-09-01-gits-database-internals-iv-distributed-synchronization/)
- [Git’s database internals V: scalability](https://github.blog/2022-09-02-gits-database-internals-v-scalability/)

## 1. GIT MANEGEMENT

<details><summary>Add new SSH key to your local machine. You must add your .pub key to your GitHub/GitLab settings. <strong>IMPORTANT: the ssh-agent can only use ONE ssh key at a time.</strong></summary>

Check for existing SSH keys. (Private key: ed25519; public key: ed25519.pub)

```sh
ls -al ~/.ssh
```

Generate SSH key with your e-mail account.

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Start ssh-agent in the background.

```sh
eval "$(ssh-agent -s)"
```

Add your SSH private key to the ssh-agent.

```sh
ssh-add ~/.ssh/id_ed25519
```

Authenticate yourself to be able to connect to GitHub. This command attempts to ssh to GitHub.

```sh
ssh -T git@github.com
```

</details>

<details><summary>Manage multiple accounts with SSH.</summary>

Create a config file (without extensions) in your ~/.ssh/ folder. In the config file, add the following:

```md
# GitHub account

Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_github

# Gitlab account

Host gitlab.com  
 HostName gitlab.com  
 User git
IdentityFile ~/.ssh/id_rsa_gitlab
```

This configuration asks `ssh-agent` to:

- 1, Use id_rsa_github as the key for any Git URL that uses @github.com
- 2, Use the id_rsa_gitlab key for any Git URL that uses @gitlab.com

Alternatively you can remove all ssh entries from the `ssh agent`.

```sh
ssh-add -D
```

Then add the relevant ssh key at start of every session.

```sh
ssh-add ~/.ssh/id_rsa
```

<strong>IMPORTANT! When managing multiple accounts, changing the ssh key is not enough. You must ensure the git config user name and email is exactly what you want -- either locally or globally.</strong>

</details>

<details><summary>Manage multiple accounts with GPG.</summary>

THIS SECTION IS WORK IN PROGRESS!

</details>

<details><summary>Set user settings. If used without --global, it only changes the config of the local repository. The local config file will overwrite the global settings.</summary>

Set your full name.

```sh
git config --global user.name "your_name"
```

Set your e-mail you use with your GitHub/GitLab account.

```sh
git config --global user.email "your_email@example.com"
```

Set the username of your GitHub/GitLab account

```sh
git config --global credential.username "your_account_name_here"
```

In the event of a merge conflict, Git will launch a "merge tool." Replace `tool-name` with tool you would like to use, e. g. Vim or kdiff3.

```sh
git config --global merge.tool <tool-name>
```

Make the default branch (after git init) main, instead of master. Though you can write anything instead of main.

```sh
git config --global init.defaultBranch main
```

Make git to always sign commits by default with your GPG key.

```sh
git config --global commit.gpgsign true
```

</details>

<details><summary>Check the SHA-1 ID or OIDs of files, file-types and contents.</summary>

Output file parameters, mainly the SHA-1 IDs. `path` can be: HEAD, HEAD:filename, branch-name, etc. You can the ID of blobs, trees and commits.

```sh
git rev-parse <path>
```

Output the contents of one or more objects.

```sh
git cat-file -p <SHA-1 ID>
```

Output the type of one or more objects.

```sh
git cat-file -t <SHA-1 ID>
```

</details>

## 2. GIT BASICS

<details><summary>Clone a git repository to your local machine.</summary>

Clone a git repository to local machine via SSH. SSH key must be present for authentication.

```sh
git clone <ssh-link>
```

Clone a git repository to local machine via HTTPS.

```sh
git clone <https-link>
```

Clone a git repository to local machine via HTTPS and downloads all submodules according to submodule configuration.

```sh
git clone --recurse-submodules <https-link>
```

</details>

<details><summary>Initialize and manage a git repository in a local directory.</summary>

Initialize a git repository in a local directory.

```sh
git init
```

See the status of the current repository. Lists if there are any changes or any actions required.

```sh
git status
```

Add specific files to the stating area.

```sh
git add filename
```

Patch level of a file. Select which changes should be staged from the specific file.

```sh
git add -p filename
```

Add every changed files to the stating area.

```sh
git add .
```

Commit changes to git with a message about the changes. Messages should be clear and descriptive. Only combine changes from the same topic in a single commit. If you are not sure, you should be able to read a commit message like this: <strong>if this commit is applied, it will "your message".</strong>

```sh
git commit -m "message"
```

Add the changes, then commit to git with a message. Only if the changed files were committed previously.

```sh
git commit -am "message"
```

Change the message of the last commit. Never change commit history of a remote repository (because the commit hash will change too)!

```sh
git commit --amend -m "message"
```

The new commit is a fix for the specified earlier commit and is automatically placed in the old commit's position (similar to squash).

```sh
git commit --fixup commit-hash
```

</details>

<details><summary>Working with stashes.</summary>

View all stashes in your current branch.

```sh
git stash list
```

The git stash command takes your uncommitted changes (both staged and unstaged), saves them away for later use, and then reverts them from your working copy.

```sh
git stash
```

Same as git stash, but includes the untracked, ignored (e. g. newly added) files too.

```sh
git stash -u
```

It's good practice to annotate your stashes with a description.

```sh
git stash save "message"
```

Popping your stash removes the changes from your stash and reapplies them to your working copy.

```sh
git stash pop
```

By default, git stash pop will re-apply the most recently created stash: stash@{0}. Choose which stash to re-apply by passing its identifier as the last argument.

```sh
git stash pop stash@{2}
```

Reapplies the changes to your working copy, but keeps them in your stash.

```sh
git stash apply
```

If the changes on your branch diverge from the changes in your stash, you may run into conflicts when popping or applying your stash. Instead, you can use git stash branch to create a new branch to apply your stashed changes to.

```sh
git stash branch <branch-name> stash@{1}
```

Delete a particular stash.

```sh
git stash drop stash@{1}
```

Delete all of your stashes.

```sh
git stash clear
```

</details>

## 3. GIT HISTORY

<details><summary>Check the commit history in your current repository.</summary>

Show commit history.

```sh
git log
```

A concise version of the history is displayed.

```sh
git log --oneline
```

--decorate will format the commit history.

```sh
git log --decorate
```

The --graph option draws an ASCII graph representing the branch structure of the commit history. This is commonly used in conjunction with the --oneline and --decorate commands to make it easier to see which commit belongs to which branch.

```sh
git log --graph
```

Prevent git log from displaying these merge commits by passing the --no-merges flag.

```sh
git log --no-merges
```

</details>

<details><summary>Search filters for git log (e. g. git log --after="2021-07-15"). Search filters can be used in conjunction with each other.</summary>

Show logs only after the specified date.

```sh
git log --after="YYYY-MM-DD"
```

Show logs only before the specified date.

```sh
git log --before="YYYY-MM-DD"
```

Show logs where the message has the specified word in it. grep accepts regular expressions too.

```sh
git log --grep="word"
```

Show commits from the specified author only.

```sh
git log --author="name"
```

Show all commits that are in the main branch, but not those that are in feature/login. So with git log, other branches commit histories can be viewed also. Comes in handy, when you are working in another branch, and want to see what happened in the main branch since your branch was split.

```sh
git log feature/login..main
```

</details>

<details><summary>Shortlog, blame and reflog.</summary>

A special version of git log. It groups each commit by author and displays the first line of each commit message.

```sh
git shortlog
```

The git blame command is used to examine the contents of a file line by line and see when each line was last modified and who the author of the modifications was.

```sh
git blame <filename>
```

This command manages the information recorded in the reflogs. Reference logs, or "reflogs", record when the tips of branches and other references were updated in the local repository. Reflogs are useful in various Git commands, to specify the old value of a reference.

```sh
git reflog
```

</details>

<details><summary>Creating and managing tags. By default, git push will not push tags. Tags have to be explicitly passed to git push.</summary>

List stored tags in a repo.

```sh
git tag
```

Creates a tag. Replace `tagname` with a semantic identifier to the state of the repo at the time the tag is being created. A common pattern is to use version numbers like git tag v1.4.

```sh
git tag <tagname>
```

Creates an annotated tag. Annotated tags are stored as full objects in the Git database. To reiterate, They store extra meta data such as: the tagger name, email, and date. Annotated tagged should be signed with a GPG key.

```sh
git tag -a <tagname>
```

Creates a signed tag.

```sh
git tag -s <tagname>
```

Delete the specified tag.

```sh
git tag -d <tagname>
```

</details>

## 4. GIT BRANCHES

<details><summary>See, set and modify branches in your current repository.</summary>

See all the branches in the current repository.

```sh
git branch
```

Create a new branch. Typical long-term branches are: main, dev. Typical short-term branches are: feature, hotfix, release.

```sh
git branch <branch-name>
```

Rename the current branch to the new branch-name.

```sh
git branch -m <branch-name>
```

Delete the selected branch.

```sh
git branch -d <branch-name>
```

With reflog, even deleted branches' hash number can be seen. This way, the deleted branches can be recovered.

```sh
git branch develop <branch-hash>
```

Change between branches in the current repository.

```sh
git checkout <branch-name>
```

View the differences between the current branch and another branch (branch-name).

```sh
git diff <branch-name>
```

</details>

<details><summary>Merge branches or undo merges.</summary>

Join two or more development histories together. It will create a merged commit. In sub-branch, git merge main will update the sub-branch to the latest main branch code.

```sh
git merge
```

NOTE: <strong>pull requests are the preferred method for merging on remote repositories.</strong> Push the sub-branch to remote repository as upstream. THEN on GitHub, Compare & Pull Request. Write a comment. Wait for feedback. Finally, Merge pull request. The sub-branch can be deleted.

```sh
git push -u origin <branch-name>
```

Undo the merge that e. g. caused a merge conflict.

```sh
git merge --abort
```

Run merge conflict resolution tools to resolve merge conflicts.

```sh
git mergetool
```

</details>

## 5. GIT REMOTE REPOSITORIES

<details><summary>Set or change the remote repository connections.</summary>

List the remote connections you have to other repositories.

```sh
git remote -v
```

Create a new connection to a remote repository. After adding a remote, you’ll be able to use `name` as a convenient shortcut for `url` in other Git commands.

```sh
git remote add <name> ssh://git@<github-url>/<project>.git
```

Change the url of remote repository if it was set earlier.

```sh
git remote set-url <name> ssh://git@<github-url>/<project>.git
```

Remove the connection to the remote repository called `name`.

```sh
git remote rm <name>
```

Rename a remote connection from `old-name` to `new-name`.

```sh
git remote rename <old-name> <new-name>
```

</details>

<details><summary>Pull, fetch or push from/to a remote repository.</summary>

Push changes to a remote repository from the local machine.

```sh
git push
```

Push changes to a remote repository from the local machine. Set the origin main as the upstream branch, so from now on, git push in itself is enough.

```sh
git push -u origin main
```

By default, git push will not push tags. Tags have to be explicitly passed to git push. To push multiple tags simultaneously pass the `--tags` option.

```sh
git push origin <tag-name>
```

Pull changes from a remote repository to local machine.

```sh
git pull
```

If the error is fatal: refusing to merge unrelated histories on every try, this will allow the pull. However, it should be used carefully. You would only want to merge unrelated histories if the two repositories indeed share very little in common. For example, each repository contributes a different set of files during the merges.

```sh
git pull --allow-unrelated-histories
```

Download commits, files and refs from a remote repository into your local repo. Fetching is what you do when you want to see what everybody else has been working on. Git isolates fetched content from existing local content; it has absolutely no effect on your local development work. Fetched content has to be explicitly checked out using the git checkout command.

```sh
git fetch
```

Only fetch the specified branch.

```sh
git fetch <remote> <branch>
```

It is the best utility for cleaning outdated branches. It will connect to a shared remote repository remote and fetch all remote branch refs. It will then delete remote refs that are no longer in use on the remote repository.

```sh
git fetch --prune
```

</details>

## 6. GIT UNDO CHANGES

<details><summary>Reset changes from a different commit. git reset is a simple way to undo changes that haven’t been shared with anyone else.</summary>

Reset the changes from the last commit to the one before it. The number is the number of commits before the last one.

```sh
git reset HEAD~1
```

git log shows the hash number of each commit. This hash number can be added to be able to reset the project version to a specific stage.

```sh
git reset <commit-hash>
```

Reset to a previous version and delete all local changes.

```sh
git reset --hard <commit-hash>
```

</details>

<details><summary>Restore deleted files or a specific version of files.</summary>

Restore a deleted file.

```sh
git restore <filename>
```

Approve or discard chunks of changes in a file.

```sh
git restore -p <filename>
```

Restore a specific version of a specific file.

```sh
git restore --source <commit-hash> <filename>
```

Revert the changes of the specified commit. A revert is an operation that takes a specified commit and creates a new commit which inverses the specified commit. Reverting undoes a commit by creating a new commit. This is a safe way to undo changes, as it has no chance of re-writing the commit history.

```sh
git revert <commit-hash>
```

Cherry-picking. Select a commit from another branch and move to the active branch. Should do a git reset --hard HEAD~1 on the other branch to clean it from unnecessary changes.

```sh
git cherry-pick <commit-hash>
```

</details>

<details><summary>Rebase branches.</summary>

Merge the selected branch to the current branch, rewriting current branch's commit history. Only use for local development when cleaning up short-term branches.

```sh
git rebase <branch-name>
```

Undo the rebase that e. g. caused a merge conflict.

```sh
git rebase --abort
```

Start an interactive rebase session three commits back from the current version.

```sh
git rebase -i HEAD~3
```

<details><summary>Options in an interactive rebase session.</summary>

Change the commit message of the previous commit.

```md
reword
```

Remove/delete the selected commit.

```md
drop
```

Meld the selected commit into a previous commit.

```md
squash
```

</details>

</details>

## 7. GIT OPTIMIZATION

<details><summary>Creating a commit-graph.</summary>

This creates a commit-graph in your working repository. The commit-graph acts as a specialized query index. Note: running manually (not via `git maintenance run`) will only create the commit-graph for the current state of the repository and must be updated manually. The `--changed-paths` option adds Bloom filters, so that `git log` and `git blame` skips treesame commits without parsing any trees.

```sh
git commit-graph write --reachable --changed-paths
```

</details>

<details><summary>Working with submodules.</summary>

Download external library to project in a clean way. The project only has the submodule configurations, not the actual submodules.

```sh
git submodule add "https-link"
```

Download the submodule data according to the submodule configuration.

```sh
git submodule update --init --recursive
```

</details>

<details><summary>Working with subtrees.</summary>

THIS SECTION IS WORK IN PROGRESS!

</details>

## 8. GIT IN PRACTICE

<details><summary>How to move a full Git repository. (Source: https://www.atlassian.com/git/tutorials/git-move-repository)</summary>

1. Create a local repository in the temp-dir directory using:

```sh
git clone <url to ORI repo> temp-dir
```

2. Go into the temp-dir directory.

3. To see a list of the different branches in ORI do:

```sh
git branch -a
```

4. Checkout all the branches that you want to copy from ORI to NEW using:

```sh
git checkout <branch-name>
```

5. Now fetch all the tags from ORI using:

```sh
git fetch --tags
```

6. Before doing the next step make sure to check your local tags and branches using the following commands:

```sh
git tag
git branch -a
```

7. Now clear the link to the ORI repository with the following command:

```sh
git remote rm origin
```

8. Now link your local repository to your newly created NEW repository using the following command:

```sh
git remote add origin <url to NEW repo>
```

9. Now push all your branches and tags with these commands:

```sh
git push origin --all
git push --tags
```

10. You now have a full copy from your ORI repo.

</details>

<details><summary>Oh shit, I did something terribly wrong, please tell me git has a magic time machine!?! (Source: https://ohshitgit.com/)</summary>

You can use this to get back stuff you accidentally deleted, or just to remove some stuff you tried that broke the repo, or to recover after a bad merge, or just to go back to a time when things actually worked.

```sh
git reflog
git reset HEAD@{index}
```

</details>

<details><summary>Oh shit, I committed and immediately realized I need to make one small change! (Source: https://ohshitgit.com/)</summary>

1. Make your change.

```sh
git add . # or add individual files
git commit --amend --no-edit
```

2. Now your last commit contains that change! <strong>Warning: You should never amend commits that have been pushed up to a public/shared branch! Only amend commits that only exist in your local copy or you're gonna have a bad time.</strong>

</details>

<details><summary>Oh shit, I accidentally committed something to master that should have been on a brand new branch! (Source: https://ohshitgit.com/)</summary>

1. Create a new branch from the current state of master.

```sh
git branch <some-new-branch-name>
```

2. Remove the last commit from the master branch.

```sh
git reset HEAD~ --hard
git checkout <some-new-branch-name>
```

3. Your commit lives in this branch now. <strong>Note: this doesn't work if you've already pushed the commit to a public/shared branch.</strong>

</details>

<details><summary>Oh shit, I accidentally committed to the wrong branch! (Source: https://ohshitgit.com/)</summary>

1. Change to the correct branch.

```sh
git checkout <name-of-the-correct-branch>
```

2. Grab the last commit to main.

```sh
git cherry-pick main
```

3. Delete it from main.

```sh
git checkout main
git reset HEAD~ --hard
```

</details>
