---
title: "The flow of git"
tags: [git]
---

# What is git?
Git is a content-addressable filesystem. Great. What does that mean? It means that at the core of Git is a simple key-value data store. 
What this means is that you can insert any kind of content into a Git repository, for which Git will hand you back a unique key you can
use later to retrieve that content. [1]

The above section is taken from Git Internals. This isn't article about the basics of git, or another tutorial, since there
are plenty of them on the internet. As I'm learning this subject myself, in this article I discuss few internal parts of git.

# So how git works?
Git uses objects to store your data. There are couple types of objects:
- blob: the actual content of the file
- tree: object represents a folder and metadata of the files in it (permissions, file names)
- commit: can be thought of as a snapshot of the project content (all files, folders, etc.) at the time of the commit

These objects represented by SHA-1 hashes which are used as keys to the actual object in the database. (the repository)
You can basically retrieve ANY object using its SHA-1. When you move between branches, you basically change your working
directory to point to a different commit. A branch is just a reference to a commit. That's it.

HEAD is a special reference, that points to a branch, which points to a commit. HEAD is special because there is only
one head, and it is always pointing to the current project you work on.

# High overview of Git sections
On a local machine, Git can be broken down to 3 areas.

- Working area: the current content of the project on your project folder (locally)
- Staging (Index): changes in the project that would be included in the next commit (modify / add / remove of files and folders)
- Repository: the database which holds all your project history

The working area is the least important one. Every time you move between commits, this folder is wiped and the new content
is copied into it. (not always, but most of the time).

Understanding how Git works is important step to feel comfort with it. Let's discuss what each of the basics git commands do,
and how it effect the above 3 areas.

- add: add files to staging area. This moves files from working area to staging, and has no impact on the repository.
- rm: remove files from staging or entierly from the project. This can be useful if you want to remove a file from a commit,
and add it on a later stage. It can be used with --cached option, which removes it from staging but keeps it on the working area,
or with --force which deletes it from both areas. This also has no impact on the repository.
- commit: copies files from staging to the repository, creating a new commit which points to the previous one. Then it moves
the branch reference to the new commit.
- checkout: this command moves the repository to point on a different branch (commit), and then copies its content to
the working and staging areas. (so we are aligned, and have a clear working folder)


# The stash area
Often, when you’ve been working on part of your project, things are in a messy state and you want to switch branches for a bit to work
on something else. The problem is, you don’t want to do a commit of half-done work just so you can get back to this point later. 
The answer to this issue is the git stash command. (-u or --include-untracked to include untracked files in stash)

To list all the stashed stages you have, use `git stash list`
Then when you want to go back to the state you were, just do `git stash apply` to retrieve it.

# Git reset
reset is a very powerful tool, that is not so obvious what it actually solely does by its name. But after you get used to it,
it makes sense. So, what does `git reset` do?

Reset has three options: --soft, --mixed and --hard.
It also does two things. One of them is common to all the three options.
First, reset *moves the branch to reference another commit*. The second part depends on what option you passed to git.

- `--mixed` is the default option. This is *copying files from the remote repository to the index area*, so it basically removes
all the files to be committed from the index, and you are now aligned with the remote repository.
- `--soft` This does nothing else, just the first task (moving branch commit)
- `--hard` is *copying from the repository to BOTH index and working area*, so all three stages are now aligned.

If you want to update the REMOTE repository as well, avoiding a new commit and actually changing the project history,
you can use:

'''python
git reset --hard <commit-hash> 
git push -f <remote> <local branch>:<remote branch> 
'''

*CAUTION*: do not use this if anyone else pulled the changes you are reverting. This will prevent them from adding new commits without
manually resolve conflicts. If other people already has the code you want to remove, it is better to revert the changes locally and then
push it as a new commit.

Another way to revert changes using reset, is this:

'''python
$ git reset --hard <commit>
$ git reset --soft @{1}  # (or ORIG_HEAD)
$ git commit
'''

The first command resets the branch reference to a specific commit and copies all the files to the working + index areas.
The second, resets just the branch reference, back to the latest commit in the branch. This won't copy anything.
And then just commit, which creates a new commit on top of the existing one. This reverts back the branch to a specific snapshot,
without changing history (because its a new commit).

# Git revert
`git revert <commit>` removes the changes made by a specific commit, and creates a new commit without these changes.
there is an option `--no-commit` to prevent automatically create a new commit after this command. This would be useful when
reverting couple commits, and you want to combine this action into a single commit (multiple commits reverts -> single new commit on top of HEAD)

This command works by committing the opposite of the reverted commit. (this is not UNDO operation)
Should not be used when reverting a merge commit; as this gets really confusing to track the history of the project.
When reverting a merge, a better option is to use the *reset* tool and move the reference back before the merge happened.

# Git checkout <branch / commit> [file]
The git checkout command serves three distinct functions: checking out files, checking out commits, and checking out branches.
When checking out a commit (branch), git copies all the project from the repository to the working directory + the index areas.
You are now aligned with the remote repo.

If you need to reset a single file, git reset won't let you, because it always has to update the branch reference. When passing a file,
git reset will return an error. So how do we reset just a single file?
`git checkout branch file_name` will copy just this file from the remote repository to your working + index areas. Reverting any changes
made to it prior the command.

git checkout is a read only operation. No harm can be done using it. When checking out a file from an older commit, you can commit it again
with a new commit, basically reverting it back.

checking out an older commit does change your HEAD reference. This will cause a status named 'detached HEAD', which means your HEAD reference
is not aligned with your branch reference. The two references pointing to a different commits.

# Git amend
amending is a tool to change the latest commit on the branch. You do your changes, and add `--amend` to your commit command.
This causes git to COPY the latest commit, add your new changes and combine them together to a new commit.
It then changes the branch reference to point to the new commit. The old one eventually gets garbage collected, since no reference
points to it.

[1] https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
[2] http://eagain.net/articles/git-for-computer-scientists/