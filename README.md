Git & GitHub CLI — Personal Command Documentation

A practical reference for managing Git repositories, commit identity, GitHub authentication, repositories, and collaborator access from the VS Code Git Bash terminal on macOS.

1. Check the Current Git Repository
Show the connected GitHub repository
git remote -v
Show only the origin URL
git remote get-url origin
Show the current branch
git branch --show-current
Check repository status
git status
2. Check Git User / Commit Identity

Git stores the author and committer information inside each commit.

Check current repository's username
git config user.name
Check current repository's email
git config user.email
Check global Git identity
git config --global user.name
git config --global user.email
Set identity for the current repository
git config user.name "Srabon"
git config user.email "YOUR_GITHUB_EMAIL"

Use the email address associated with your GitHub account.

Verify
git config user.name
git config user.email
3. Check Commit Author & Committer

To inspect the latest commit:

git show -s --format=fuller HEAD

You'll see something similar to:

Author:     Srabon <email@example.com>
Commit:     Srabon <email@example.com>
Author vs Committer
Author → person who originally created the changes
Committer → person who created the commit object

Usually they are the same person.

4. Fix the Latest Commit Identity

If the latest commit was created using the wrong Git identity:

git commit --amend --reset-author --no-edit

Then check:

git show -s --format=fuller HEAD

If the commit was already pushed and you amended it, the commit hash changed.

You may need:

git push --force-with-lease

Use force-push carefully, especially on shared repositories.

5. Handle a Non-Fast-Forward Push

If:

git push

returns:

rejected (non-fast-forward)

and your local and remote branches have diverged:

git fetch origin

Then:

git pull --rebase origin main

Check:

git status

Then:

git push origin main

If there is a merge/rebase conflict, stop and resolve the conflict before continuing.

6. GitHub CLI (gh)

gh means GitHub CLI — GitHub Command Line Interface.

It allows you to manage GitHub directly from your terminal.

Check if installed
gh --version

If you get:

bash: gh: command not found

install it with Homebrew:

brew install gh

Verify:

gh --version
7. Log Into GitHub CLI

Start authentication:

gh auth login

Recommended options:

GitHub.com
HTTPS
Login with a web browser

After authentication:

gh auth status

Make sure the account shown is:

srabon-chowdhury-47
8. List GitHub Repositories
Basic repository list
gh repo list --limit 1000
9. Get Detailed Repository Permissions

This was particularly useful for identifying collaboration repositories:

gh api user/repos --paginate --jq '.[] | "\(.full_name)\t\(.permissions.admin)\t\(.permissions.push)\t\(.permissions.pull)"'

Example:

ajoy-dutta/project1    false   true    true
srabon-chowdhury-47/project2    true    true    true
Meaning
admin = true

Generally means you have administrative control.

admin = false
push = true
pull = true

Means you have write/collaborator access but are not an administrator.

10. List Only Non-Admin Repositories
gh api user/repos --paginate --jq '.[] | select(.permissions.admin == false) | .full_name'

This is useful for finding repositories where someone else has given you access.

11. Leave a GitHub Collaboration

If someone added you as a collaborator and you want to leave their repository:

gh api --method DELETE "/repos/OWNER/REPO/collaborators/srabon-chowdhury-47"

Example:

gh api --method DELETE "/repos/ajoy-dutta/example-repo/collaborators/srabon-chowdhury-47"
Important

This:

gh api --method DELETE ...

removes your access only.

It does NOT delete the repository.

12. Bulk Leave Collaborations
Preview before removing anything

For example, to list all non-admin repositories owned by ajoy-dutta:

gh api user/repos --paginate --jq '.[] | select(.owner.login == "ajoy-dutta" and .permissions.admin == false) | .full_name'

Always review the output before running a bulk operation.

13. Leave All ajoy-dutta Collaborations Except heavenauto

This was the specific operation used in this session.

Preview
gh api user/repos --paginate --jq '.[] | select(.owner.login == "ajoy-dutta" and .permissions.admin == false and .full_name != "ajoy-dutta/heavenauto") | .full_name'

If the output is correct, execute:

gh api user/repos --paginate --jq '.[] | select(.owner.login == "ajoy-dutta" and .permissions.admin == false and .full_name != "ajoy-dutta/heavenauto") | .full_name' | while read repo; do
  echo "Leaving $repo"
  gh api --method DELETE "/repos/$repo/collaborators/srabon-chowdhury-47"
done
Result

This:

Leaves all ajoy-dutta/* collaborations
Keeps ajoy-dutta/heavenauto
Keeps your own repositories
Does not delete any repositories
14. Leave One Specific Collaboration

Example:

gh api --method DELETE "/repos/Mithi2002/AMS/collaborators/srabon-chowdhury-47"

Again, this only removes your access.

15. Repository Deletion — DO NOT Confuse This With Leaving
Leaving a collaboration
gh api --method DELETE "/repos/OWNER/REPO/collaborators/srabon-chowdhury-47"

Removes your access.

Deleting your own repository
gh repo delete OWNER/REPO --yes

Permanently deletes the repository.

These are completely different operations.

16. GitHub CLI Delete Permission

If you intentionally want to delete one of your own repositories and receive:

This API operation needs the "delete_repo" scope.

authorize the permission:

gh auth refresh -h github.com -s delete_repo

Then:

gh repo delete OWNER/REPO --yes

⚠️ Use this only when you actually intend to permanently delete your own repository.

17. Useful Git Commands Cheat Sheet
Task	Command
Repository URL	git remote -v
Current branch	git branch --show-current
Repository status	git status
Recent commits	git log --oneline -10
Git username	git config user.name
Git email	git config user.email
Fetch remote	git fetch origin
Rebase from remote	git pull --rebase origin main
Push main	git push origin main
Latest commit details	git show -s --format=fuller HEAD
Amend latest commit	git commit --amend --reset-author --no-edit
18. Useful GitHub CLI Commands Cheat Sheet
Task	Command
Check gh	gh --version
Login	gh auth login
Check account	gh auth status
List repositories	gh repo list --limit 1000
Repository details	gh repo view OWNER/REPO
Clone repository	gh repo clone OWNER/REPO
GitHub API	gh api ...
Leave collaboration	gh api --method DELETE "/repos/OWNER/REPO/collaborators/USERNAME"
Delete own repository	gh repo delete OWNER/REPO --yes
Request delete permission	gh auth refresh -h github.com -s delete_repo
19. Safety Rules
Before bulk operations

Always preview the repositories first:

gh api user/repos --paginate --jq '.[] | .full_name'
For collaboration cleanup

Use:

gh api --method DELETE ...

not:

gh repo delete ...
Before deleting a repository

Make absolutely sure you intend to permanently remove it.

Before force-pushing

Prefer:

git push --force-with-lease

instead of:

git push --force
Before account-level GitHub operations

Check:

gh auth status

and make sure you're using the correct GitHub account.

Quick Workflow

For future GitHub collaboration cleanup:

# 1. Check GitHub account
gh auth status

# 2. Find repositories where you are not admin
gh api user/repos --paginate --jq '.[] | select(.permissions.admin == false) | .full_name'

# 3. Preview a specific owner's collaborations
gh api user/repos --paginate --jq '.[] | select(.owner.login == "OWNER" and .permissions.admin == false) | .full_name'

# 4. Leave one repository
gh api --method DELETE "/repos/OWNER/REPO/collaborators/srabon-chowdhury-47"

# 5. Verify
gh api user/repos --paginate --jq '.[] | .full_name'

Key distinction to remember:

git  → manages Git repositories, commits, branches, remotes
gh   → manages GitHub, repositories, authentication, permissions, API
