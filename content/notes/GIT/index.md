---
title: GIT
weight: 700
menu:
  notes:
    name: Cheat sheet
    identifier: notes-git-cheatsheet
    weight: 70
---

<div style="display: block; width: 100%; max-width: none;">
<!-- Intro: -->
{{< note title="Version control:" >}}
What is Version Control?  
Version control systems are tools that manage changes made to files and directories in a project. They allow you to keep track of what you did when, undo any changes you decide you don't want, and collaborate at scale with others. This cheat sheet focuses on one of the most popular one, Git.
{{< /note >}}
<!-- Definitions: -->
{{< note title="Definitions:" >}}
*Basic definition*
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
  
*More advanced definition*
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
$ brew install git
```
On Linux
```bash
$ sudo apt-get install git
```
Check if installation successful (On any platform)
```bash
$ git --version
```
{{< /note >}}

<!-- Setup: -->
{{< note title="Configuration:" >}}
If you are working in a team on a single repo, it is important for others to know who made certain changes to the code. So, Git allows you to set user credentials such as name, email, etc..
*basic information*
- Configure your email
```bash
$ git config user.email [your.email@domain.com©
```
- Configure your name
```bash
$ git config user.name [your-name]
```
*Important tags*
to determine the scope of configurations, Git lets you use tags to determine the scope of the information you’re using during setup
- Local directory, single project (this is the default tag)
```bash
$ git config --local user.email “my_email@example.com
```
- All git projects under the current user
```bash
$ git config --global user.email “my_email@example.com
```
- For all users on the current machine
```bash
$ git config --system user.email “my_email@example.com”
```
*List configurations*
List all key-value configurations
```bash
$ git config --list
```
- Get the value of a single key
```bash
$ git config --get <key>
```
*Setting aliases*
- Create an alias named gc for the “git commit” command
```bash
$ git config --global alias.gc commit
```
Example usage:
```bash
$ gc -m "New commit"
```
- Create an alias named ga for the “git add” command
```bash
$ git config --global alias.ga add
```
{{< /note >}}

<!-- Sources: -->
{{< note title="sources:" >}}
[DatacampCheatsheet](https://media.datacamp.com/legacy/image/upload/v1656573882/Marketing/Blog/git_cheat_sheet.pdf)
[GitlabCheatsheet](https://about.gitlab.com/images/press/git-cheat-sheet.pdf)
{{< /note >}}

</div>