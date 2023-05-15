# 6.5.7. 明确的 NUMA 最佳化

假如所有节点上的所有执行绪都需要存取相同的memory区域时，所有的本地memory与亲和性规则都无法帮上忙。当然，简单地将执行绪的数量限制为直接连接到memory节点的处理器所能支援的数量是可能的。不过，这并没有善用 SMP NUMA 机器，因此并不是个实际的选项。

若是所需的资料是唯读的，有个简单的解法：复制（replication）。每个节点能够得到它自己的资料副本，这样就不必进行节点之间的存取。这样做的程序码看起来可能像这样：

```c
void *local_data(void) {
  static void *data[NNODES];
  int node =
    NUMA_memnode_self_current_idx();
  if (node == -1)
    /* Cannot get node, pick one. */
    node = 0;
  if (data[node] == NULL)
    data[node] = allocate_data();
  return data[node];
}
void worker(void) {
  void *data = local_data();
  for (...)
    compute using data
}
```

在这段程序中，函式 `worker` 借由一次 `local_data` 的呼叫来取得一个资料的本地副本的指标来进行准备。接著它继续执行使用这个指标的回圈。`local_data` 函式保存一个已经被分配的资料副本的列表。每个系统拥有有限的memory节点，所以带有各节点memory副本的指标的阵列大小是受限的。来自 libNUMA 的 `NUMA_memnode_system_count` 函式回传这个数字。若是给定节点的memory还没被分配给当前节点（由 `data` 在 `NUMA_memnode_self_current_idx` 呼叫所回传的索引位置的空指标来识别），就分配一个新的副本。

重要的是要意识到，如果在 `getcpu` 系统呼叫之后，执行绪被排程在另一个连接到不同memory节点的 CPU 上时，是不会发生什么可怕的事情的。[^43]它只代表在 `worker` 中使用 `data` 变数的存取，存取另一个memory节点上的memory。这直到 `data` 被重新计算为止会拖慢程序，但就这样。系统核心总是会避免不必要的、每颗 CPU 执行伫列的重新平衡。若是这种转移发生，通常是为了一个很好的理由，并且在不久的未来不会再次发生。

当处理中的memory区域是可写入的，事情会更为复杂。在这种情况下，单纯的复制是行不通的。根据具体情况，或许有一些可能的解法。

举例来说，若是可写入的memory区域是用来累积（accumulate）结果的，首先为结果被累积的每个memory节点建立一个分离的区域。接著，当这项工作完成时，所有的每节点的memory区域会被结合以得到全体的结果。即使运作从不真的停止，这项技术也能行得通，但中介结果是必要的。这个方法的必要条件是，结果的累积是无状态的（stateless）。即，它不会依赖先前收集起来的结果。

不过，拥有对可写入memory区域的直接存取总是比较好的。若是对memory区域的存取数量很可观，那么强迫系统核心将处理中的memory分页迁移到本地节点可能是个好点子。若是存取的数量非常高，并且在不同节点上的写入并未同时发生，这可能有帮助。但要留意，系统核心无法产生奇蹟：分页迁移是一个复制操作，并且以此而论并不便宜。这个成本必须被分期偿还。



[^43]: 使用者层级的 `sched_getcpu` 介面是使用 `getcpu` 系统呼叫来实作的。后者不该被直接使用，并且有一个不同的介面。

