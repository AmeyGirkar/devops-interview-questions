# Top 100 Git Commands: The Ultimate Reference Guide

## Basic Configuration and Setup
1. **git config**
   - Sets configuration values for your Git installation
   - Examples:
     ```
     git config --global user.name "Your Name"
     git config --global user.email "your.email@example.com"
     git config --list  # Show all configurations
     ```

2. **git init**
   - Initializes a new Git repository
   - Creates a new .git subdirectory in your current project
   - Usage: `git init`

3. **git clone**
   - Creates a copy of an existing Git repository
   - Syntax: `git clone <repository-url>`
   - Examples:
     ```
     git clone https://github.com/user/repository.git
     git clone https://github.com/user/repository.git my-folder  # Clone to specific folder
     git clone --depth 1  # Shallow clone with only the most recent commit
     ```

## Basic Snapshotting

4. **git add**
   - Adds file contents to the index (staging area)
   - Variations:
     ```
     git add <filename>       # Add specific file
     git add .                # Add all new and changed files
     git add -A               # Stage all files (new, modified, deleted)
     git add -p               # Interactively choose parts of files to stage
     ```

5. **git commit**
   - Records changes to the repository
   - Examples:
     ```
     git commit -m "Commit message"
     git commit -a -m "Commit all tracked changes"
     git commit --amend       # Modify the most recent commit
     git commit -am "Message" # Add and commit in one step
     ```

6. **git status**
   - Shows the status of changes as untracked, modified, or staged
   - Usage: `git status`
   - Options:
     ```
     git status -s            # Short format
     git status -b            # Show branch information
     ```

7. **git diff**
   - Shows changes between commits, commit and working tree, etc.
   - Variations:
     ```
     git diff                 # Changes not yet staged
     git diff --staged        # Changes between staging and last commit
     git diff HEAD            # Changes in working tree since last commit
     git diff <commit1> <commit2>  # Differences between two commits
     ```

## Branching and Merging

8. **git branch**
   - Lists, creates, or deletes branches
   - Examples:
     ```
     git branch                   # List branches
     git branch <branch-name>     # Create a new branch
     git branch -d <branch-name>  # Delete a branch
     git branch -m <old-name> <new-name>  # Rename a branch
     git branch -a                # List all branches (local and remote)
     ```

9. **git checkout**
   - Switches branches or restores working tree files
   - Examples:
     ```
     git checkout <branch-name>     # Switch to a branch
     git checkout -b <new-branch>   # Create and switch to a new branch
     git checkout -- <file>         # Discard changes in working directory
     git checkout <commit> <file>   # Restore file from a specific commit
     ```

10. **git merge**
    - Joins two or more development histories together
    - Examples:
      ```
      git merge <branch-name>        # Merge specified branch into current branch
      git merge --no-ff <branch>     # Create merge commit even if fast-forward possible
      git merge --squash <branch>    # Merge changes without commit history
      ```

11. **git rebase**
    - Reapplies commits on top of another base tip
    - Examples:
      ```
      git rebase <branch>            # Rebase current branch onto another
      git rebase -i HEAD~3           # Interactive rebase of last 3 commits
      git rebase --continue          # Continue after resolving conflicts
      ```

## Remote Repository Interactions

12. **git remote**
    - Manages set of tracked repositories
    - Examples:
      ```
      git remote add <name> <url>    # Add a new remote repository
      git remote -v                  # List all remotes with URLs
      git remote show <remote>       # Show information about a remote
      git remote rename <old> <new>  # Rename a remote
      ```

13. **git fetch**
    - Downloads objects and refs from another repository
    - Examples:
      ```
      git fetch origin              # Fetch from default remote
      git fetch --all                # Fetch from all remotes
      git fetch origin <branch>      # Fetch specific branch
      ```

14. **git pull**
    - Fetches from and integrates with another repository or a local branch
    - Examples:
      ```
      git pull origin master         # Pull changes from master branch
      git pull --rebase              # Pull with rebase instead of merge
      git pull -r origin <branch>    # Shorthand for rebase pull
      ```

15. **git push**
    - Updates remote refs using local refs
    - Examples:
      ```
      git push origin master         # Push to master branch
      git push -u origin <branch>    # Push and set upstream
      git push --all origin          # Push all branches
      git push origin --delete <branch>  # Delete remote branch
      ```

## Inspection and Comparison

16. **git log**
    - Shows commit logs
    - Examples:
      ```
      git log                        # Basic log
      git log -n <number>            # Show last n commits
      git log --oneline              # Compact log format
      git log --graph                # Show branch graph
      git log -p <file>              # Show changes for specific file
      ```

17. **git show**
    - Shows various types of objects
    - Examples:
      ```
      git show <commit>              # Show details of a specific commit
      git show HEAD                  # Show last commit
      git show --name-only <commit>  # Show only changed files
      ```

18. **git blame**
    - Shows who changed what and when in a file
    - Usage: `git blame <file>`
    - Options:
      ```
      git blame -L 10,15 <file>      # Blame specific line range
      git blame -w <file>            # Ignore whitespace changes
      ```

## Undoing Changes

19. **git reset**
    - Resets current HEAD to the specified state
    - Examples:
      ```
      git reset HEAD <file>          # Unstage a file
      git reset --soft HEAD~1        # Undo last commit, keep changes staged
      git reset --hard HEAD          # Discard all local changes
      ```

20. **git revert**
    - Creates a new commit that undoes all of the changes made in a specified commit
    - Usage: `git revert <commit-hash>`

## Advanced Operations

21. **git stash**
    - Temporarily stores modified, tracked files
    - Examples:
      ```
      git stash                      # Stash current changes
      git stash list                 # List all stashes
      git stash apply                # Apply most recent stash
      git stash pop                  # Apply and remove stash
      ```

22. **git tag**
    - Create, list, delete or verify tags
    - Examples:
      ```
      git tag <tagname>              # Create lightweight tag
      git tag -a v1.0 -m "Message"   # Create annotated tag
      git tag -l                     # List tags
      ```

## Additional Powerful Commands

23. **git cherry-pick**
    - Apply the changes introduced by some existing commits
    - Usage: `git cherry-pick <commit-hash>`

24. **git reflog**
    - Shows a log of all ref updates (HEAD, branches)
    - Usage: `git reflog`

25. **git bisect**
    - Use binary search to find the commit that introduced a bug
    - Interactive debugging tool for finding problematic commits

## Bonus Advanced Commands

26. **git submodule**
    - Initialize, update or inspect submodules
    - Examples:
      ```
      git submodule add <repository> <path>
      git submodule update --init
      ```

27. **git archive**
    - Create an archive of files from a named tree
    - Usage: `git archive --format=zip HEAD > project.zip`

## Pro Tips
- Always use descriptive commit messages
- Commit frequently and in logical chunks
- Use branches for feature development
- Pull before you push to avoid conflicts
- Learn to use interactive rebase for clean history

## Learning Resources
- Official Git Documentation: https://git-scm.com/docs
- GitHub Learning Lab: https://lab.github.com
- Git Cheat Sheet: https://training.github.com/downloads/github-git-cheat-sheet.pdf

*Note: This guide covers the most common and useful Git commands. Git has many more advanced features and options.*
