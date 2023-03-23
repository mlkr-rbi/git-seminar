# Git RBI seminar

## References

Git documentation
<https://git-scm.com/doc>

Pro Git
<https://git-scm.com/book/en/v2>

Git Immersion
<https://gitimmersion.com/>

Getting started with GitHub documentation
<https://docs.github.com/en/get-started>

Git from the bottom up
<http://ftp.newartisans.com/pub/git.from.bottom.up.pdf>

Git Data Transport Commands - visualization of a Git dataflow
<https://blog.osteele.com/2008/05/my-git-workflow/>

Git Cheat Sheet
<https://about.gitlab.com/images/press/git-cheat-sheet.pdf>

Informative git prompt for bash and fish
<https://github.com/magicmonty/bash-git-prompt>

## Basic git setup

Global (user-level) git config is in `~/.gitconfig`:
```
cat ~/.gitconfig
vim ~/.gitconfig
```
System-level git config is in `/etc/gitconfig`, accessible with `--system`, and you need root privileges to write there!
User-level git config is in `~/.gitconfig` or `~/.config/git/config`, accessible with `--global`.
Local-level git config is in `.git/config`, accessible with `--local`.

Each (more specific) level overrides values in the previous (more general) level!

List git config:
```
git config --list
git config --list --global
git config --list --local
git config --list --show-origin
```

Setup global user name and email:
```
git config --global user.name "Matija Piskorec"
git config --global user.email "matija.piskorec@gmail.com"
```

Use silent less (without beep sound) when running git diff and similar paging commands:
```
git config --global core.pager 'less -q'
```

In your git config, add following line in order to use `git hist` command instead of `git log`:
```
[alias]  
    hist = log --pretty=format:\"%h %ad | %s%d [%an]\" \--graph --date=short
```

To set a different name for the default branch, instead of `master`:
```
git config --global init.defaultBranch main
```

The default branch on Github is named `main` instead of `master` as before.

Git help is accessed with:
```
git help
git add -h
git commit -h
```

## Initialize local repository

To initialize local git repository run:
```
git init
```

Check the local `.git` folder created!

Check the current status:
```
git status
```

## Clone a remote repository

Clone a repository:
```
git clone [REPOSITORY URL]
```

Clone into a specific folder:
```
git clone [REPOSITORY URL] [FOLDER NAME]
```

## Basic git usage

Create some file:
```
echo "This is my first file." > file1.txt
```

Add file to staging area:
```
git add file1.txt
git status
```

Remove file from staging area:
```
git rm --cached file1.txt 
```

Commit the changes to the file:
```
git commit -m "Initial commit"
```

See the commit history:
```
git log 
git hist 
```

Adding some more changes:
```
echo "This is second line in first file" >> file1.txt
echo "This is my second file" > file2.txt
```

Making two separate commits:
```
git add file1.txt
git commit -m "Update file1.txt"
git add file2.txt
git commit -m "Add file2.txt"
```

Git focuses on changes, not files:
```
echo "Second line in file2.txt" >> file2.txt
git add file2.txt
echo "Third line in file2.txt" >> file2.txt
git status
git add .
git commit -m "Update file2.txt"
```

Difference between repository and tree (all objects are stored in `.git/objects`):
```
git cat-file -t fbf38
git cat-file -p fbf38
git hash-object -w file1.txt
```

Making some damage and reverting to HEAD commit (changes working directory):
```
rm file1.txt file2.txt
git status
ls -la
git checkout .
git status
```

Notice that deleting works fine, Git can stage the removal, but renaming a file stages a deletion of the original file, while the new file is considered untracked:
```
mv file1.txt file1-new.txt
```

Git checkout will now unstage the deletion of the original file but the new file will remain untracked:
```
git checkout .
git status
```

Just check what will be changed:
```
git checkout -- .
```

If you want to rename a file you should use a `git mv` command:
```
git mv [OLD FILENAME] [NEW FILENAME]
```

Unstageing (leaves working directory unchanged, the two files are still deleted!):
```
git reset HEAD
git status 
```

To return the original files:
```
git checkout .
```

Reset the last commit. Commit will be deleted and the changes will re-enter the staging area (useful if you are not satisfied with your commit and want to change it):
```
git reset HEAD~1
```

Revert both staged and unstaged changes in the current directory:
```
git reset --hard HEAD
```

To discard unstaged changes on a specific file:
```
echo "Update to file1" >> file1.txt
git status
git restore file1.txt
```

To unstage currently staged changes for a file:
```
echo "Update to file1" >> file1.txt
git add file1.txt
git status
git restore --staged [FILE]
```

View changes in unstaged files as compared to the latest commit:
```
echo "Update to file1" >> file1.txt
git diff file1.txt
```

**Pro tip**: Use `git diff` before a commit to see which changes to commit in which files! This is useful if you want to separate changes in different commits.

View changes in staged files as compared to the latest commit:
```
git add file1.txt
git diff --cached file1.txt
```

Checkout an older commit (you can get has of a commit with `git log` or `git hist`):
```
git hist
git checkout 77b8c6c
```

To switch back to the original branch just checkout the branch name:
```
git branch -a
git checkout master
```

## Branches

Basic branching and merging from official Git documentation:
<https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging>

List all branches, both local and remote:
```
git branch -a
```

Create and checkout a new branch:
```
git checkout -b feature01
```

This is a shorthand for:
```
git branch feature01
git checkout feature01
```

You do some changes and commit them, then you want to go back to the master branch and start doing some changes:
```
echo "This is file3.txt" >> file3.txt
git add file3.txt
git commit -m "Add file3.txt"
git checkout master
```

Note: If you have some uncommited changes, sometimes you will not be able to do this - Git won't let you switch branches if it cannot apply changes you did to the working tree of another branch. In that case, either commit all of your changes, or you stash them (see section on stashing). You can do a simple example here, returning to the feature01 branch, reseting a commit and trying to switch branches:
```
git checkout feature01
git reset HEAD~1
git checkout master
```

In this case it will work - the file3.txt will reamin in your working tree even if you switch to the master branch. You can return to the feture01 branch and recommit the file3.txt:
```
git checkout feature01
git add file3.txt
git commit -m "Add file3.txt"
git checkout master
```

Notice that there is no file3.txt in your working tree once you switched to the master branch! This is because you commited the change in your feature01 branch.

Once you are able to checkout the master branch, created the hotfix branch, did your hotfix and tested everything, you are ready to merge the branch hotfix back to the master branch:
```
git checkout -b hotfix
echo "Hotfix to file2.txt" >> file2.txt
git add file2.txt
git commit -m "Hotfix to file2.txt"
git checkout master
git merge hotfix
```

If there were no other commits on the master branch, the merge will be a simple fast-forward. The master branch will now contain changes from the hotfix branch. We can delete the hotfix branch:
```
git branch -d hotfix
```

Now lets checkout feature01 branch and do some changes:
```
git checkout feature01
cat "Add feature01 to file2.txt" >> file2.txt
git checkout master
```

You will not be able to switch to master branch because your local uncommited changes are conflicting with the file2.txt from the master branch (remember, there you applied a hotfix to file2.txt).

## Stashes

To make a quick stash which stashes all staged and unstaged changes to all files (but not untracked and ignored files!):
```
git stash
```

List all stashes:
```
git stash list
```

Show the list of changes in the last stash:
```
git stash show
```

Show exact changes in the last stash:
```
git stash show -p
```

Reapply previously stashed changes and remove them from the stash:
```
git stash pop
```

So lets put our changes to the stash and switch to the master branch, and then back to feature01 branch:
```
git stash
git checkout master
git checkout feature01
```

Lets pop the changes from the stash, add them to the staging area and commit them:
```
git pop
git add file2.txt
git commit -m "Add feature01 to file2.txt"
```

## Merging

Now lets checkout master branch and merge our changes from the feature01 branch:
```
git checkout master
git merge feature01
```

The automatic merge will fail! The changes we did on file2.txt (hotfix and feature01) are conflicting! We should open the file2.txt, resolve the merge conflict and then commit the desired version. This is how file2.txt looks like:
```
This is my second file.
Second line in file2.txt
Third line in file2.txt
<<<<<<< HEAD
Hotfix to file2.txt
=======
Add feature01 to file2.txt
>>>>>>> feature01
```

We can decide to keep both changes, so we edit the file to look like this:
```
This is my second file.
Second line in file2.txt
Third line in file2.txt
Hotfix to file2.txt
Add feature01 to file2.txt
```

We can add the change and commit file2.txt:
```
git add file2.txt
git commit -m "Merge feature01 and resolve conflict"
```

A special merge commit will be created - it has two parents!

## Remote repositories on Github

Go to Github and create a repository. You now have a remote repository which you can either clone to your local machine, or to which you can push changes from your local repository. There are three ways to do it...

First way, simply clone the empty repository locally. You can do it in two ways - over HTTPS or over SSH:
```
git clone git@github.com:mlkr-rbi/git-seminar.git
git clone https://github.com/mlkr-rbi/git-seminar.git
```

If the repository is public you will be able to clone it over HTTPS, but you will not be able to push any changes! This is because Github enforces SSH authentification. We will explain how to setup your SSH credentials.

To clone it in a specific directory with different name than `git-seminar`:
```
git clone https://github.com/mlkr-rbi/git-seminar.git rbi-seminar/
```

Second way, initalize a local git repository, set the remote origin and push some changes:
```
echo "# git-seminar" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:mlkr-rbi/git-seminar.git
git push -u origin main
```

Again you can choose to set the origin to the HTTPS link (instead of SSH), but then you will not be able to push any changes.

Third way, if you already have a local repository on your local machine, you can just set the remote origin and push all commits to your Github remote repository:
```
git remote add origin git@github.com:mlkr-rbi/git-seminar.git
git branch -M main
git push -u origin main
```

I show a combination of these approaches.

Clone the remote repository over HTTPS:
```
git clone https://github.com/mlkr-rbi/git-seminar.git
```

Commit some files to the repository:
```
cd git-seminar/
echo "# git-seminar" >> README.md
git add README.md
git commit -m "Initial commit"
git remote show origin
git remote set-url origin git@github.com:mlkr-rbi/git-seminar.git
git remote show origin
```

We are not able to even view the remote repository once we changed from HTTSP to SSH! We have to setup an SSH key:

Generate key pair that you will use for authentification (when prompted for a password, you can use the one you usually use for Github for example):
```
ssh-keygen -t rsa ~/.ssh/github_rsa
```

This will generate two keys in your `~/.ssh/` folder: 

    `~/.ssh/github_rsa` - private key which you should never share with anyone!
    `~/.ssh/github_rsa.pub` - a public key which you can share with the world

Login to your Github account, go to `Settings`, then to `SSH and GPG keys`, and choose the option `Add SSH key`. Paste your `~/.ssh/github_rsa.pub` there.

Run ssh-agent which will allow you to put keys in cache so you don't have to enter keyphrase:
```
eval `ssh-agent`
```

Add key to cache for eight hours and then delete it (so that someone cannot steal the key from memory):
```
ssh-add -t 8h ~/.ssh/github_rsa
```

List all keys that are added to the ssh-agent:
```
ssh-add -L
```

Delete all keys currently in ssh-agent:
```
ssh-add -D
```

You will have to do this every time you reboot. But once you add the key and type in the passphrase you won't have to do it again during the session.

Test your ssh connection to Github:
```
ssh -T git@github.com
```

If everything is ok and Github responds with your username, your key is set and you should be able to push without typing your password. If your account has proper permissions for the remote repository you should be able to see the remote origin:
```
git remote show origin
```

You can now push changes to the repository:
```
git push -u origin main
```

For demonstration, lets clone a repository again to simulate another user, and do some changes:
```
git clone git@github.com:mlkr-rbi/git-seminar.git git-seminar-v2/
echo "Some changes..." >> README.md
git add README.md
git commit -m "Update README"
git push origin main
```

Lets go to the first repository and pull the changes from Github:
```
cd ../git-seminar/
git pull
```

Lets create a local branch and push it to Github:
```
git checkout -b matija
echo "This is a line from Matija branch!" >> README.md
git add README.md
git commit -m "Add one line to README from Matija branch"
git push -u origin matija
```

Lets switch to main branch and pull changes from matija branch, then push them to main branch on Github:
```
git checkout main
git pull origin matija
git push origin main
```

Here we pulled changes from the remote branch matija, but we can also pull changes from the local branch matija:
```
git checkout main
git merge matija
git push origin main
```

