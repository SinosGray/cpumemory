# 6.5.5. 查询节点资讯

`get_mempolicy` 介面能用以查询关于一个给定地址的 NUMA 状态的各种事实。

```c
#include <numaif.h>
long get_mempolicy(int *policy,
             const unsigned long *nmask,
             unsigned long maxnode,
             void *addr, int flags);
```

当 `get_mempolicy` 以 `0` 作为 `flags` 参数呼叫时，关于地址 `addr` 的策略资讯会被储存在由 `policy` 指到的word、以及由 `nmask` 指到的节点的bit遮罩中。若是 `addr` 落在一段已经被指定一个 VMA 策略的定址空间范围中，就回传关于这个策略的资讯。否则，将会回传关于任务策略或者 –– 必要的话 –– 系统预设策略的资讯。

若是 `flags` 中设定 `MPOL_F_NODE`、并且管理 `addr` 的策略为 `MPOL_INTERLEAVE`，那么 `policy` 所指到的word为要进行下一次分配的节点索引。这个资讯能够潜在地用来设定打算要在新分配的memory上运作的一条执行绪的亲和性。这可能是实现逼近的一个比较不昂贵的方法，尤其是在执行绪仍未被建立的情况。

`MPOL_F_ADDR` 旗标能用来检索另一个完全不同的资料项目。假如使用这个旗标，`policy` 所指到的word为已经为包含 `addr` 的分页分配memory的memory节点索引。这个资讯能用来决定可能的分页迁移、决定哪条执行绪能够最有效率地运作在memory位置上、还有更多更多的事情。

一条执行绪正在使用的 CPU –– 以及因此用到的memory节点 –– 比起它的memory分配还要更加变化无常。在没有明确请求的情况下，memory分页只会在极端的情况下被移动。一条执行绪能被指派给另一个 CPU，作为重新平衡 CPU 负载的结果。关于当前 CPU 与节点的资讯可能因而仅在短期内有效。排程器会试著将执行绪维持在同一个 CPU 上，甚至可能在相同的核上，以最小化由于冷cache（cold cache）造成的效能损失。这表示，查看当前 CPU 与节点的资讯是有用的；只要避免假设关联性不会改变。

libNUMA 提供两个介面，以查询一段给定虚拟定址空间范围的节点资讯：

```c
#include <libNUMA.h>
int NUMA_mem_get_node_idx(void *addr);
int NUMA_mem_get_node_mask(void *addr,
                           size_t size,
                           size_t __destsize,
                           memnode_set_t *dest);
```

`NUMA_mem_get_node_mask` 根据管理策略，在 `dest` 中设置代表所有分配（或者可能分配）范围 [`addr`, `addr`+`size`) 中的分页的memory节点的bit。`NUMA_mem_get_node` 只看地址 `addr`，并回传分配（或者可能分配）这个地址的memory节点的索引。这些介面比 `get_mempolicy` 还容易使用，而且应该是首选。

当前正由一条执行绪使用的 CPU 能够使用 `sched_getcpu` 来查询（见 6.4.3 节）。使用这个资讯，一支程序能够使用来自 libNUMA 的 `NUMA_cpu_to_memnode` 介面来确定 CPU 本地的memory节点（们）：

```c
#include <libNUMA.h>
int NUMA_cpu_to_memnode(size_t cpusetsize,
                        const cpu_set_t *cpuset,
                        size_t memnodesize,
                        memnode_set_t *
                        memnodeset);
```

对这个函式的一次呼叫会设置所有对应于任何在第二个参数指到的集合中的 CPU 本地的memory节点的bit。就如同 CPU 资讯本身，这个资讯直到机器的配置改变（例如，CPU 被移除或新增）时才会是正确的。

`memnode_set_t` 物件中的bit能被用在像 `get_mempolicy` 这种低阶函式的呼叫上。使用 libNUMA 中的其它函式会更加方便。反向映射能透过下述函式取得：

```c
#include <libNUMA.h>
int NUMA_memnode_to_cpu(size_t memnodesize,
                        const memnode_set_t *
                        memnodeset,
                        size_t cpusetsize,
                        cpu_set_t *cpuset)
```

在产生的 `cpuset` 中设置的bit为任何在 `memnodeset` 中设置的bit所对应的memory节点本地的那些 CPU。对于这两个介面，程序开发者都必须意识到，资讯可能随著时间改变（尤其是使用 CPU 热插拔的情况）。在许多情境中，在输入的bit集中只会设置单一个bit，但举例来说，将 `sched_getaffinity` 呼叫检索到的整个 CPU 集合传递到 `NUMA_cpu_to_memnode`，以确定哪些memory节点能够被执行绪直接存取到，也是有意义的。

