# Git Scenarios and Solutions

## 1. Merge Conflict Resolution
**Question: You are working in a team and while merging a feature branch into the main branch, you encounter a merge conflict. How would you resolve it?**

When encountering a merge conflict:
1. First, I would run `git status` to identify the conflicting files.
2. Open the conflicting files and look for the conflict markers (<<<<<<, =======, >>>>>>>).
3. Manually edit the files to resolve the conflicts, carefully reviewing the changes from both branches.
4. Remove the conflict markers, keeping the desired code.
5. Stage the resolved files using `git add <filename>`.
6. Complete the merge by running `git commit`.

If the conflicts are complex, I might:
- Consult with the team members who made the conflicting changes
- Use a merge tool like `git mergetool` for a more visual conflict resolution
- Ask a teammate to help review the merge

## 2. Reverting a Buggy Commit
**Question: You realize that a commit you pushed to the main branch introduced a bug. How would you revert that commit without affecting subsequent commits?**

To revert a commit without affecting subsequent commits:
```bash
# Find the specific commit hash of the problematic commit
git log

# Create a new commit that undoes the changes from the specific commit
git revert <commit-hash>
```

Key considerations:
- `git revert` creates a new commit that undoes the changes
- This approach is safe for commits already pushed to a shared repository
- It maintains the commit history and doesn't alter existing commits
- If the revert is complex, review the changes carefully before committing

## 3. Squashing Commits
**Question: Your team prefers a clean commit history. Before merging a feature branch into the main branch, how would you squash multiple commits into one?**

To squash multiple commits before merging:
```bash
# Interactive rebase, going back the number of commits you want to squash
git rebase -i HEAD~4  # Squash last 4 commits

# In the interactive editor:
# - Keep the first commit
# - Change 'pick' to 'squash' for subsequent commits
# - Edit the commit message as desired
```

Alternative method when merging:
```bash
# When merging a feature branch
git merge --squash feature-branch
git commit
```

## 4. Managing Code Reviews
**Question: How would you manage a code review process using GitHub to ensure high code quality and collaboration within your team?**

Implementing a robust code review process:
1. Branch Management
- Create feature branches for each task
- Use descriptive branch names
- Never commit directly to main/master branch

2. Pull Request Workflow
- Open a pull request for code review
- Ensure CI checks pass before review
- Require at least one approval from a team member
- Use GitHub's review features:
  * Add inline comments
  * Request specific changes
  * Approve or request modifications

3. Review Checklist
- Code quality and readability
- Adherence to coding standards
- Test coverage
- Performance considerations
- Security implications

4. Automated Checks
- Set up branch protection rules
- Require status checks to pass
- Mandate code review approvals
- Use linters and automated testing

## 5. Removing Sensitive Information
**Question: A teammate accidentally committed sensitive information (e.g., passwords, API keys) to the repository. How would you remove this information from the Git history?**

To remove sensitive data from Git history:
```bash
# Use BFG Repo-Cleaner or git-filter-repo
git filter-repo --invert-paths --path path/to/sensitive/file

# Or using git-filter-branch (older method)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive/file" \
  --prune-empty --tag-name-filter cat -- --all

# Force push to remote (with caution)
git push origin --force --all
```

Crucial steps:
- Immediately rotate exposed credentials
- Inform team about the sensitive data exposure
- Ensure all team members re-clone the repository

## 6. Rebase vs. Merge
**Question: Your team is debating whether to use rebase or merge for integrating changes from one branch to another. What are the pros and cons of each approach?**

### Merge
Pros:
- Preserves complete history
- Shows exactly when and how branches were integrated
- No risk of rewriting shared history

Cons:
- Can create complex, non-linear history
- Merge commits can clutter repository history

### Rebase
Pros:
- Creates a linear, clean commit history
- Helps maintain a more readable timeline
- Reduces unnecessary merge commits

Cons:
- Rewrites commit history
- Can cause issues if used on shared branches
- Potential for merge conflicts during rebase

Recommendation:
- Use merge for public/shared branches
- Use rebase for local feature branches

## 7. Continuous Integration with GitHub Actions
**Question: How would you set up a CI pipeline using GitHub Actions to automatically run tests and build the project on every push and pull request?**

Sample GitHub Actions workflow:
```yaml
name: CI

on: [push, pull_request]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '14'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Build project
      run: npm run build
```

## 8. Forking Workflow for Open Source Contributions
**Question: You want to contribute to an open-source project hosted on GitHub. Describe the forking workflow and how you would manage your contributions.**

1. Fork the original repository
2. Clone your forked repository
3. Create a new branch for your contribution
4. Make and commit changes
5. Push changes to your fork
6. Open a pull request to the original repository
7. Respond to feedback and iteration

Key practices:
- Keep your fork synchronized with the original repository
- Use `git remote add upstream <original-repo-url>`
- Regularly fetch and merge upstream changes

## 9. Tagging Releases
**Question: How would you create and manage tags in Git for marking release versions of your project?**

```bash
# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag with message
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tags to remote
git push origin v1.0.0  # Specific tag
git push origin --tags  # All tags
```

## 10. Git Submodules
**Question: Your project depends on another Git repository. How would you include this dependency using Git submodules and manage it?**

```bash
# Add a submodule
git submodule add <repository-url> <path>

# Initialize and update submodules
git submodule update --init --recursive

# Update submodule to latest commit
git submodule update --remote
```

Considerations:
- Submodules link to specific commits
- Requires careful management
- Team must be familiar with submodule workflow
