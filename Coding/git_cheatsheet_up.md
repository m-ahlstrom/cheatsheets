Set the current branch upstream.

```sh
git branch --set-upstream-to <remote-branch>
```

Before deleting files, a dry run should be run with the `-n` flag, because 1, `git clean` is undoable and 2, the dry run will only output what the command would do, but won't actually run it.

```sh
git clean -n
```

Delete untracked files in the working repository. `git clean` will only run with the `-f` or `--force` option, this is a built-in safety mechanism.

```sh
git clean -f
```

List references available in a remote repository along with the associated commit IDs.

```sh
git ls-remote
```

<details><summary>Find bugs with bisecting.</summary>

Start bisecting.

```sh
git bisect start
```

Declare the current commit bad. Git assumes you mean HEAD by default.

```sh
git bisect bad
```

If you reach a commit where the bug is not present, you can declare it good.

```sh
git bisect good
```

The bisect tool repeats until resetting.

```sh
git bisect reset
```

</details>

<details><summary>How to split a commit into multiple ones.</summary>

1. Find the commit hash with `git reflog`. Then start and interactive rebase session.

```sh
git rebase -i <commit-hash>
```

2. In the rebase edit screen, find the line with the commit that you want to split and replace `pick` with `edit`. Then reset the state to the previous commit

```sh
git reset HEAD~
```

3. Now, you can create different commits. Finally, finish the rebase session.

```sh
git rebase --continue
```

4. If anything goes sideways, you can easily start over with the `git rebase --abort`.

</details>
