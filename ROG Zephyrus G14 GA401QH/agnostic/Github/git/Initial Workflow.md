*/path/to/files/pushing/to/repo*
```bash
git init    # Initialize new git repo
git remote add [REMOTE-NAME] [URL/SSH ADDRESS]
git remote -v    # Verify remote
git add .        # Add files to the staging commit
git status       # Confirm the staging commit
git commit -m [MSG]    # Commit changes in local repo
git push --set-upstream [REMOTE-NAME] [LOCAL-BRANCH-NAME]
git pull [REMOTE-NAME] [REMOTE-BRANCH-NAME]
```