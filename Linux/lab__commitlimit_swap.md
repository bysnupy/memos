## How CommitLimit affect the Swap occurrence is not so much?

### Test patterns

No.|Pattern
-|-
1| CommitLimit < Total Memory, and allocated nearly total memory.
2| CommitLimit > Total Memory, and allocated nearly total memory.
3| After disabling overcommit feature, allocated more memories than CommitLimit.
4| CommitLimit < Total Memory, and allocated nearly total memory as virtual memory only.
5| The overcommit and heuristic overcommit take turns to allocate more memory than physical memory.

### Environment
Item|Value
-|-
OS|CentOS 7.3.1611
Kernel|3.10.0-514.26.1.el7.x86_64

#### 1. CommitLimit < Total Memory, and allocated nearly total memory

Process memory size

Item|Value
-|-
VSZ|8132076 kB
RSS|7280144 kB

/proc/meminfo and related kernel parameters

Item|Value
-|-
MemTotal|8175660 kB
SwapTotal|1048572 kB
SwapFree|272400 kB
CommitLimit|5136400 kB
Committed_AS|8356328 kB
vm.overcommit_ratio|50
vm.overcommit_memory|0
oom_score|844

#### 2. CommitLimit > Total Memory, and allocated nearly total memory.

Process memory size

Item|Value
-|-
VSZ|8132076 kB
RSS|7209032 kB

/proc/meminfo and related kernel parameters

Item|Value
-|-
MemTotal|8175660 kB
SwapTotal|1048572 kB
SwapFree|218912 kB
CommitLimit|9224232 kB
Committed_AS|8341268 kB
vm.overcommit_ratio|100
vm.overcommit_memory|0
oom_score|843

#### 3. After disabling overcommit feature, allocated more memories than CommitLimit.

After taking effect as vm.overcommit_memory = 2, this pattern has failed memory allocation.

#### 4. CommitLimit < Total Memory, and allocated nearly total memory as virtual memory only.

Process memory size

Item|Value
-|-
VSZ|8199004 kB
RSS|42080 kB

/proc/meminfo and related kernel parameters

Item|Value
-|-
MemTotal|8175660 kB
SwapTotal|1048572 kB
SwapFree|1048572 kB
CommitLimit|5136400 kB
Committed_AS|8531808 kB
vm.overcommit_ratio|50
vm.overcommit_memory|0
oom_score|5

#### 5. The overcommit and heuristic overcommit take turns to allocate more memory than physical memory + swap memory.

Both vm.overcommit_memory = 0 and vm.overcommit_memory = 1 are failed to allocating memory, but each result was different.
vm.overcommit_memory = 0 has terminated before allocating the memory, but vm.overcommit_memory = 1 was killed by OOM killer (it has allocated the memory probably)

### Conclusion

CommitLimit was not a single factor causing swap paging, instead Commited_As ratio against Total memory was more important one.
but if overcommit feature is disabled, CommitLimit is real limitation regardless total memory size.
