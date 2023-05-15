# 5.3. 被发布的资讯

系统核心透过 `sys` 虚拟档案系统（sysfs）将处理器cache的资讯发布在

`/sys/devices/system/cpu/cpu*/cache`

在 6.2.1 节，我们会看到能用来查询不同cache大小的介面。这里重要的是cache的拓朴。上面的目录包含了列出 CPU 拥有的不同cache资讯的子目录（叫做 `index*`）。档案 `type`、`level`、与 `shared_cpu_map` 是在这些目录中与拓朴有关的重要档案。一个 Intel Core 2 QX6700 的资讯看起来就如表 5.1。

<figure>
  <table>
    <tr>
      <th colspan="2"></th>
      <th><code>type</code></th>
      <th><code>level</code></th>
      <th><code>shared_cpu_map</code></th>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu0</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000003</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu1</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000003</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu2</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>0000000c</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu3</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>0000000c</td>
    </tr>
  </table>
  <figcaption>表 5.1：Core 2 CPU cache的 <code>sysfs</code> 资讯</figcaption>
</figure>

这份资料的意义如下：

* 每颗处理器核[^25]拥有三个cache：L1i、L1d、L2。
* L1d 与 L1i cache没有被任何其它的处理器核所共享 –– 每颗处理器核有它自己的一组cache。这是由 `shared_cpu_map` 中的bit图（bitmap）只有一个被设置的bit所暗示的。
* `cpu0` 与 `cpu1` 的 L2 cache是共享的，正如 `cpu2` 与 `cpu3` 上的 L2 一样。

若是 CPU 有更多cache阶层，也会有更多的 `index*` 目录。

<figure>
  <table>
    <tr>
      <th colspan="2"></th>
      <th><code>type</code></th>
      <th><code>level</code></th>
      <th><code>shared_cpu_map</code></th>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu0</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu1</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu2</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu3</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu4</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu5</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu6</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu7</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000080</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000080</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000080</td>
    </tr>
  </table>
  <figcaption>表 5.2：Opteron CPU cache的 <code>sysfs</code> 资讯</figcaption>
</figure>

对于一个四槽、双核的 Opteron 机器，cache资讯看起来如表 5.2。可以看出这些处理器也有三种cache：L1i、L1d、L2。没有处理器核共享任何阶层的cache。这个系统有趣的部分在于处理器拓朴。少了这个额外资讯，就无法理解cache资料。`sys` 档案系统将这个资讯摆在下面这个档案

`/sys/devices/system/cpu/cpu*/topology`

表 5.3 显示了在 SMP Opteron 机器的这个阶层里头的令人感兴趣的档案。

<figure>
  <table>
    <tr>
      <th></th>
      <th><code>physical_<br />package_id</code></th>
      <th><code>core_id</code></th>
      <th><code>core_<br />siblings</code></th>
      <th><code>thread_<br />siblings</code></th>
    </tr>
    <tr>
      <td><code>cpu0</code></td>
      <td rowspan="2">0</td>
      <td>0</td>
      <td>00000003</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>cpu1</code></td>
      <td>1</td>
      <td>00000003</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>cpu2</code></td>
      <td rowspan="2">1</td>
      <td>0</td>
      <td>0000000c</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>cpu3</code></td>
      <td>1</td>
      <td>0000000c</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>cpu4</code></td>
      <td rowspan="2">2</td>
      <td>0</td>
      <td>00000030</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>cpu5</code></td>
      <td>1</td>
      <td>00000030</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>cpu6</code></td>
      <td rowspan="2">3</td>
      <td>0</td>
      <td>000000c0</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>cpu7</code></td>
      <td>1</td>
      <td>000000c0</td>
      <td>00000080</td>
    </tr>
  </table>
  <figcaption>表 5.3：Opteron CPU 拓朴的 <code>sysfs</code> 资讯</figcaption>
</figure>

将表 5.2 与 5.3 摆在一起，我们能够发现

* 没有 CPU 拥有 HT （`thethread_siblings` bit图有一个bit被设置）、
* 这个系统实际上共有四个处理器（`physical_package_id` 0 到 3）、
* 每个处理器有两颗核、以及
* 没有处理器核共享任何cache。

这正好与较早期的 Opteron 一致。

目前为止提供的资料中完全缺少的是，有关这台机器上的 NUMA 性质的资讯。任何 SMP Opteron 机器都是一台 NUMA 机器。为了这份资料，我们必须看看在 NUMA 机器上存在的 `sys` 档案系统的另一个部分，即下面的阶层中

`/sys/devices/system/node`

这个目录包含在系统上的每个 NUMA 节点的子目录。在特定节点的目录中有许多档案。在前两张表中描述的 Opteron 机器的重要档案与它们的内容显示在表 5.4。

<figure>
  <table>
    <tr>
      <th></th>
      <th><code>cpumap</code></th>
      <th><code>distance</code></th>
    </tr>
    <tr>
      <td><code>node0</code></td>
      <td>00000003</td>
      <td>10 20 20 20</td>
    </tr>
    <tr>
      <td><code>node0</code></td>
      <td>0000000c</td>
      <td>20 10 20 20</td>
    </tr>
    <tr>
      <td><code>node2</code></td>
      <td>00000030</td>
      <td>20 20 10 20</td>
    </tr>
    <tr>
      <td><code>node3</code></td>
      <td>000000c0</td>
      <td>20 20 20 10</td>
    </tr>
  </table>
  <figcaption>表 5.4：Opteron 节点的 <code>sysfs</code> 资讯</figcaption>
</figure>

这个资讯将所有的一切连系在一起；现在我们有个机器架构的完整轮廓了。我们已经知道机器拥有四个处理器。每个处理器构成它自己的节点，能够从 `node*` 目录的 `cpumap` 档案中的值里头设置的bit看出来。在那些目录的 `distance` 档案包含一组值，一个值代表一个节点，表示在各个节点上存取memory的成本。在这个例子中，所有本地memory存取的成本为 10，所有对任何其它节点的远端存取的成本为 20。[^26]这表示，即使处理器被组织成一个二维超立方体（见图 5.1），在没有直接连接的处理器之间存取也没有比较贵。成本的相对值应该能用来作为存取时间的实际差距的估计。所有这些资讯的准确性是另一个问题了。


[^25]: `cpu0` 到 `cpu3` 为处理器核的相关资讯来自于另一个将会简短介绍的地方。

[^26]: 顺带一提，这是不正确的。这个 ACPI 资讯明显是错的，因为 –– 虽然用到的处理器拥有三条连贯的超传输连结 –– 至少一个处理器必须被连接到一个南桥上。至少一对节点必须因此有比较大的距离。

