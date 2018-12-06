# Pull remote branches
## Command References

### Track the `origin/branchname` and create `mybranch`.
~~~
$ git checkout -b mybranch origin/abranch
~~~

### Track the `origin/branchname`. 
~~~
$ git checkout --track origin/branchname
~~~

### Clone remote branch `branchname` from `example.com/origin/reponame`
~~~
git clone -b branchname example.com/origin/reponame
~~~
