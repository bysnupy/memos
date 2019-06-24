# OVS CLI quick reference

## USAGE

### port list of br0
~~~
# ovs-vsctl show
~~~

### open flow rules
~~~
# ovs-fsctl dump-flows br0 -O OpenFlow13 --no-stats
~~~
