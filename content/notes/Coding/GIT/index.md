---
title: GIT
weight: 610
menu:
  notes:
    name: GIT
    identifier: notes-git-cheatsheet
    parent: notes-coding
    weight: 61
---

<div style="display: block; width: 100%; max-width: none;">
<!-- Intro: -->
{{< note title="Version control:" >}}
# What is Version Control?  
Version control systems are tools that manage changes made to files and directories in a project. They allow you to keep track of what you did when, undo any changes you decide you don't want, and collaborate at scale with others. This cheat sheet focuses on one of the most popular one, Git.
{{< /note >}}
<!-- Definitions: -->
{{< note title="Definitions:" >}}

# Basic definition
- Local repo or repository:  
A local directory containing code and files for the project
- Remote repository:  
An online version of the local repository hosted on services like GitHub, GitLab, and BitBucket
- Cloning:  
The act of making a clone or copy of a repository in a new directory
- Commit:  
A snapshot of the project you can come back to
- Branch:  
A copy of the project used for working in an isolated environment without affecting the main project
- Git merge:  
The process of combining two branches together  
  
# More advanced definition
- .gitignore file:  
A file that lists other files you want git not to track (e.g. large data folders, private info, and any local files that shouldn’t be seen by the public)
- Staging area:  
a cache that holds changes you want to commit next
- Git stash:  
another type of cache that holds unwanted changes you may want to come back later
- Commit ID or hash:  
a unique identifier for each commit, used for switching to different save points
- HEAD (always capitalized letters):  
a reference name for the latest commit, to save you having to type Commit IDs. HEAD~n syntax is used to refer to older commits (e.g. HEAD~2 refers to the second-to-last commit)
{{< /note >}}

<!-- Install: -->
{{< note title="How to install:" >}}
On OS X or Windows — Using an installer
- Download the installer
- Follow the prompts  
On OS X — Using Homebrew
```bash
brew install git
```
On Linux
```bash
sudo apt-get install git
```
Check if installation successful (On any platform)
```bash
git --version
```
{{< /note >}}

<!-- Setup: -->
{{< note title="Configuration:" >}}
If you are working in a team on a single repo, it is important for others to know who made certain changes to the code. So, Git allows you to set user credentials such as name, email, etc..  
# basic information
- Configure your email
```bash
git config user.email [your.email@domain.com]
```
- Configure your name
```bash
git config user.name [your-name]
```
# Important tags
to determine the scope of configurations, Git lets you use tags to determine the scope of the information you’re using during setup
- Local directory, single project (this is the default tag)
```bash
git config --local user.email “my_email@example.com”
```
- All git projects under the current user
```bash
git config --global user.email “my_email@example.com”
```
- For all users on the current machine
```bash
git config --system user.email “my_email@example.com”
```
# List configurations
List all key-value configurations
```bash
git config --list
```
- Get the value of a single key
```bash
git config --get <key>
```
# Setting aliases
- Create an alias named gc for the “git commit” command
```bash
git config --global alias.gc commit
```
Example usage:
```bash
gc -m "New commit"
```
- Create an alias named ga for the “git add” command
```bash
git config --global alias.ga add
```
{{< /note >}}

<!-- Sources: -->
{{< note title="sources:" >}}
[DatacampCheatsheet](https://media.datacamp.com/legacy/image/upload/v1656573882/Marketing/Blog/git_cheat_sheet.pdf)  
[GitlabCheatsheet](https://about.gitlab.com/images/press/git-cheat-sheet.pdf)
{{< /note >}}

<!-- Basics: -->
{{< note title="Managing repos:" >}}
# Creating local repositories
- Clone a repository from remote hosts (GitHub, GitLab, DagsHub, etc.)
```bash
git clone <remote_repo_url>
```
- Initialize git tracking inside the current directory
```bash
git init
```
- Create a git-tracked repository inside a new directory
```bash
git init [dir_name]
```
- Clone only a specific branch
```bash
git clone -branch <branch_name> <repo_url>
```
- Cloning into a specified directory
```bash
git clone <repo_url> <dir_name>
```
# Managing remote repositories
- List remote repos
```bash
git remote
```
- Create a new connection called <remote> to a remote repository on servers like GitHub, GitLab, DagsHub, etc
```bash
git remote add <remote> <url_to_remote>
```
- Remove a connection to a remote repo called <remote>
```bash
git remote rm <remote>
```
- Rename a remote connection
```bash
git remote rename <old_name> <new_name>
```
{{< /note >}}

<!-- Files: -->
{{< note title="Handling files:" >}}
# Adding and removing files
- Add a file or directory to git for tracking
```bash
git add <filename_or_dir>
```
- Add all untracked and tracked files inside the current directory to git
```bash
git add .
```
- Remove a file from a working directory or staging area
```bash
git rm <filename_or_dir>
```
# Saving and working with changes
- See changes in the local repository
```bash
git status
```
- Saving a snapshot of the staged changes with a custom message
```bash
git commit -m “[Commit message]”
```
- Staging changes in all tracked files and committing with a message
```bash
git add -am “[Commit message]”
```
- Editing the message of the latest commit
```bash
git commit --amend -m “[New commit message]”
```
> # A note on stashes
> Git stash allows you to temporarily save edits you've made to your working copy so you can return to your work later. Stashing is especially useful when you are not yet ready to commit
changes you've done, but would like to revisit them at a later time.

- Saving staged and unstaged changes to stash for a later use
```bash
git stash
```
- Stashing staged, unstaged and untracked files as well
```bash
git stash -u
```
- Stashing everything (including ignored files)
```bash
git stash --all
```
- Reapply previously stashed changes and empty the stash
```bash
git stash pop
```
- Reapply previously stashed changes and keep the stash
```bash
git stash apply
```
- Dropping changes in the stash
```bash
git stash drop
```
- Show uncommitted changes since the last commit
```bash
git diff
```
- Show the differences between two commits (should provide the commit IDs)
```bash
git diff <id_1> <id_2>
```
{{< /note >}}

<!-- branches: -->
{{< note title="Branches:" >}}
- List all branches
```bash
git branch
```
```bash
git branch --list
```
```bash
git branch -a
```
- Create a new local branch named new_branch without checking out that branch
```bash
git branch <new_branch>
```
- Switch into an existing branch named <branch>
```bash
git checkout <branch>
```
- Create a new local branch and switch into it
```bash
git checkout -b <new_branch>
```
- Safe delete a local branch (prevents deleting unmerged changes)
```bash
git branch -d <branch>
```
- Force delete a local branch (whether merged or unmerged)
```bash
git branch -D <branch>
```
- Rename the current branch to <new_name>
```bash
git branch -m <new_name>
```
- Push a copy of local branch named branch to the remote repo
```bash
git push <remote_repo> branch~
```
- Delete a remote branch named branch
```bash
git push <remote_repo> :branch
```
```bash
git push <remote_repo> --delete branch
```
- Merging a branch into the main branch
```bash
git checkout main
git merge <other_branch>
```
- Merging a branch and creating a commit message
```bash
git merge --no-ff <other_branch>
```
- Compare the differences between two branches
```bash
git diff <branch_1> <branch_2>
```
- Compare a single <file> between two branches
```bash
git diff <branch_1> <branch_2> <file>
```
{{< /note >}}

<!-- Commits: -->
{{< note title="Handle commits:" >}}
# Pulling changes
- Download all commits and branches from the <remote> without applying them on the local repo
```bash
git fetch <remote>
```
- Only download the specified <branch> from the <remote>
```bash
git fetch <remote> <branch>
```
- Merge the fetched changes if accepted
```bash
git merge <remote>/<branch>
```
- A more aggressive version of fetch which calls fetch and merge simultaneously
```bash
git pull <remote>
```
# Review work with logs
- List all commits with their author, commit ID, date and message
```bash
git log
```
- List one commit per line (-n tag can be used to limit the number of commits displayed (e.g. -5))
```bash
git log --oneline [-n]
```
- Log all commits with diff information:
```bash
git log --stat
```
- Log commits after some date
```bash
git log --oneline --after=”YYYY-MM-DD"
```
- Log commits before some date
```bash
git log --oneline --before=”last year”
```

# Reverse changes

- Checking out (switching to) older commits
```bash
git checkout HEAD~2
```
- Checks out the third-to-last commit
```bash
git checkout <commit_id>
```
- Undo the latest commit but leave the working directory unchanged
You can undo as many commits as you want by changing the number after the tilde
```bash
git reset HEAD~1
```
- Discard all changes of the latest commit (no easy recovery)
Instead of HEAD~n, you can provide commit hash as well. Changes after that commit will be destroyed
```bash
git reset --hard HEAD~1
```
- Undo a single given commit, without modifying commits that come after it (a safe reset)
```bash
git revert [commit_id]
```
{{< /note >}}
</div>