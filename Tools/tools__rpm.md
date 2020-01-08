# RPM dependencies check

* Check reference dependencies for a specific package installed
~~~
# rpm -q --provides <specific package> | cut -d ' ' -f 1 | xargs rpm -q --whatrequires | sort | uniq
~~~

* Check requrement dependencies for a specific package installation
~~~
# rpm -q --requires <specific package> | cut -d ' ' -f 1 | xargs rpm -q --whatprovides | sort | uniq
~~~
