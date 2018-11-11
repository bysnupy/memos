# How to squash the commits after push the remote repository

## Command References

### Rebase
- Rebase 7 commits from HEAD 
~~~
# git rebase -i HEAD~7
~~~

### Push forcely - Ensure nobody pulled the codes before this work.
~~~
# git push origin <branch name> --force
~~~

Done. 
