# How to squash the commits after push the remote repository

## Command References

### Rebase
- Rebase 7 commits from HEAD 
~~~
# git rebase -i HEAD~7
~~~

### Rebase the merged commits
~~~
git reset --soft HEAD~1  # Keeps changes in the index
git commit               # Create a new commit, this time not a merge commit
git rebase -i HEAD~2     # Do an interactive rebase and squash the new commit
~~~

### Push forcely - Ensure nobody pulled the codes before this work.
~~~
# git push origin <branch name> --force
~~~

Done. 

