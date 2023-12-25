## Creating a New Project
### If already local project:
1. Go to GitHub → Repo::new
2. **`git init`**
3. **`git add .`**
4. **`git commit -m "Initial commit”`**
5. **`git remote add origin git@github.com:adazoulay/project_name.git`**
6. **`git push -u origin master`**
# Common Commands:
## Starting new Project
- **`git init`**: This command is used to start a new repository.
- **`git clone <repo>`**: This command is used to obtain a repository from an existing URL.
## Info
- **`git status`**: This command lists all the files that have to be committed
- **`git branch`**: This command lists all the local branches in the current repository
- **`git log`**: This command shows a list of all the commits made along with details like the author, the date, and the commit message
- **`git diff`**: This command shows the file differences which are not yet staged
    - **`git diff --staged`**: This command shows the differences between the files in the staging area and the latest version present
## Branching
- **`git branch <branch_name>`**: This command creates a new branch
- **`git checkout <branch_name>`**: This command is used to switch from one branch to another
- **`git checkout -b <branch_name>`**: This command creates a new branch and also switches to it
- **`git branch --set-upstream-to <remote>/<branch>`:** This sets the upstream for the currently checked-out local branch
    - After setting the upstream branch, **`git pull`** and **`git push`** without arguments will default to that branch
- **`git branch -d my-feature-branch`**: Deletes a branch
	- Use **`-D`** to force if there are unmerged changes
	- Make sure you are on a different branch when running this
## Managing Changes
### Adding Changes
- **`git add <file>`:** This command adds a file to the staging area
    - **`git add .`** adds all new and changed files in the current directory and subdirectories to the staging area
- **`git commit -m "<message>"`**: This command records or snapshots the file permanently in the version history with an associated message
    - **`git commit -a`**: The **`-a`** flag automatically stages all modified and deleted files, but not new files, before doing the commit.
- **`git push <remote> <branch>`**: This command sends the committed changes of a branch to a remote repository
    - **`git push -u <remote> <branch>`**: The **`-u`** flag sets the upstream repository for the current branch. This allows you to use **`git pull`** without any arguments to update the current branch
    - **`<remote>`** is the name of the remote repository (e.g., **`origin`**), and **`<branch>`** is the name of the branch (e.g., **`main`** or **`master`**)
### Removing Changes
- **`git rm <file>`**: This command is used to delete the file from your working directory and stage the removal of the file for the next commit: the file will be removed from the Git repository as well
- **`git revert <commit>`**: This command creates a new commit that undoes the changes made in the specified commit
- **`git reset <file>`**: This command unstages the file, but it preserves the file contents.
    - **`git reset --hard`**: This command is used to discard all changes in the working directory and staging area and to revert to the last commit.
- **`git checkout -- <file>`:** Discard all changes to a file since the last commit and reset it back to the state it was in at the last commit
    - Similar to **`git reset <file>`** but in addition to unstaging the file, **`git checkout -- <file>`** also discards any changes you've made to it in your working directory
### Getting Changes
- **`git fetch <remote>`**: This command downloads commits, files, and refs from a remote repository into your local repository
    - It fetches all the changes but does not affect your local working environment
        - That is, it doesn't merge any changes into your current branch and it doesn't modify your files in the working directory
    - **`<remote>`** is usually **`origin`**, which is the default name for the remote repository
- **`git pull <remote>`**: This command fetches and merges changes on the remote server to your working directory
    - combination of **`git fetch`** and **`git merge`**
- **`git merge <branch>`**: This command merges the specified branch’s history into the current branch
### Stashing
- **`git stash save "my_stash”`:** Allows you to create a new stash with a custom name
- **`git stash list`**: Shows you all of the stashes that you've created in your repository
- **`git stash apply stash@{n}`** Allows you to apply the changes from a stash to your working directory without removing it
- **`git stash drop stash@{n}`**: Remove a specific stash without applying it
- **`git stash clear`**: Removes all stashes from the list
- **`git stash`**: This command temporarily saves changes that you don't want to commit immediately. You can apply the changes later.
- **`git stash pop`**: This command takes the changes saved with **`git stash`** and applies them to the current branch, removing them from the stash.