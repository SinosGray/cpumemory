# 6.5.2. 指定策略

`set_mempolicy` 呼叫能够用以为目前的执行绪（对系统核心来说的任务）设定任务策略。仅有目前的执行绪会受影响，而非整个行程。

```c
#include <numaif.h>
long set_mempolicy(int mode,
                   unsigned long *nodemask,
                   unsigned long maxnode);
```

`mode` 参数必须为前一节介绍过的其中一个 `MPOL_*` 常数。`nodemask` 参数指定未来分配要使用的memory节点，而 `maxnode` 为 `nodemask` 中的节点（即bit）数量。若是 `mode` 为 `MPOL_DEFAULT`，就不需要指定memory节点，并且会忽略 `nodemask` 参数。若是为 `MPOL_PREFERRED` 传递一个空指标作为 `nodemask`，则会选择本地节点。否则，`MPOL_PREFERRED` 会使用 `nodemask` 中设置的bit所对应的最低的节点编号。

对于已经分配的memory，设定策略并没有任何影响。分页不会被自动迁移；只有未来的分配会受影响。注意到memory分配与预留定址空间之间的不同：一个使用 `mmap` 建立的定址空间区域通常不会被自动分配。在memory区域上首次的读取或写入操作会分配合适的分页。若是策略在存取相同定址空间区域的不同分页之间发生改变，或者策略允许memory的分配来自不同节点，那么一个看似均匀的定址空间区域可能会被分散在许多memory节点之中。

