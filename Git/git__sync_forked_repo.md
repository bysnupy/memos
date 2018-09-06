# How to sync forked repository with origin remote one

### Command References

- Check the current remote repository to sync with.
~~~
# git remote -v
origin	https://github.com/bysnupy/openshift-docs.git (fetch)
origin	https://github.com/bysnupy/openshift-docs.git (push)
~~~

- Add the original repository for syncing as naming `upstream`
~~~
# git remote add upstream https://github.com/openshift/openshift-docs.git
~~~

- Up to date with added `upstream` repository.
~~~
# git fetch upstream
~~~

- Merge the remote `master` branch with local `master` one.
~~~
# git checkout master

# git merge upstream/master
~~~

- Finally push the local changes to my remote github repository.
~~~
# git push origin master
~~~

Done.
