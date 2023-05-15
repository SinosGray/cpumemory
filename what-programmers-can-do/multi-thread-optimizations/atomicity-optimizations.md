# 6.4.2. 最小操作的最佳化

假如多条执行绪同时修改相同的memory位置，处理器并不保证任何具体的结果。这是个为了避免在所有情况的 99.999% 中的不必要成本而做出的慎重决定。举例来说，若有个在「S」状态的memory位置、并且有两条执行绪同时必须增加它的值的时候，在从cache读出旧值以执行加法之前，执行管线不必等待cache行变为「E」状态。而是会读取目前cache中的值，并且一旦cache行变为「E」状态，新的值便会被写回去。若是在两条执行绪中的两次cache读取同时发生，结果并不如预期；其中一个加法会没有效果。

对于可能发生并行操作的情况，处理器提供 atomic 操作。举例来说，这些 atomic 操作可能在直到能以像是 atomic 地对memory位置进行加法的方式执行加法之前，不会读取旧值。除了等待其它处理器核之外，某些处理器甚至会将特定地址的 atomic 操作发给在主机板上的其它装置。这全都会令 atomic 操作变慢。

处理器厂商决定提供不同的一组 atomic 操作。早期的 RISC 处理器，与代表简化（reduced）的「R」相符，提供非常少的 atomic 操作，有时仅有一个 atomic 的bit设置与测试。[^40]在光谱的另一端，我们有提供大量 atomic 操作的 x86 与 x86-64。普遍来说可用的 atomic 操作能够归纳成四类：
* bit测试
    - 这些操作 atomic 地设置或者清除一个bit，并回传一个代表bit先前是否被设置的状态。
* 载入锁定／条件储存（Load Lock/Store Conditional，LL/SC）[^41]
    - LL/SC 操作成对使用，其中特殊的载入指令用以开始一个事务（transaction），而最后的储存仅会在这个位置没有在这段期间内被修改的情况才会成功。储存操作指出成功或失败，所以程序能够在必要时重复它的工作。
* 比较并交换（Compare-and-Swap，CAS
    - 这是个三元（ternary）操作，仅在目前值与第三个参数值相同的时候，将一个以参数提供的值写入到一个地址中（第二个参数）；
* atomic 算术
    - 这些操作仅在 x86 与 x86-64 可用，其能够在memory位置上执行算术与逻辑操作。这些处理器拥有对这些操作的非 atomic 版本的支援，但 RISC 架构则否。所以，怪不得它们的可用性是有限的。

一个处理器架构可能会支援 LL/SC 指令或 CAS 指令其一，不会两者都支援。两种方法基本上相同；它们能提供一样好的 atomic 算术操作实作，但看起来 CAS 是近来偏好的方法。其它所有的操作都能够间接地以它来实作。例如，一个 atomic 加法：

```c
int curval;
int newval;
do {
  curval = var;
  newval = curval + addend;
} while (CAS(&var, curval, newval));
```

呼叫 `CAS` 的结果指出操作是否成功。若是它回传失败（非零的值），回圈会再次执行、执行加法、并且再次尝试呼叫 `CAS`。这会重复到成功为止。这段程序值得注意的是，memory位置的地址必须以两个独立的指令来计算。[^42]对于 LL/SC，程序看起来大致相同：

```c
int curval;
int newval;
do {
  curval = LL(var);
  newval = curval + addend;
} while (SC(var, newval));
```

这里我们必须使用一个特殊的载入指令（`LL`），而且我们不必将memory位置的目前值传递给 `SC`，因为处理器知道memory位置是否曾在这期间被修改过。

<figure>
  <table>
    <tr>
      <td><pre><code>for (i = 0; i < N; ++i)
  __sync_add_and_fetch(&var,1);</code></pre></td>
      <td><pre><code>for (i = 0; i < N; ++i)
  __sync_fetch_and_add(&var,1);</code></pre></td>
      <td><pre><code>for (i = 0; i < N; ++i) {
  long v, n;
  do {
    v = var;
    n = v + 1;
  } while (!__sync_bool_compare_and_swap(&var, v,n));
}</code></pre></td>
    </tr>
    <tr>
      <th>1. 做加法并读取结果</th>
      <th>2. 做加法并回传旧值</th>
      <th>3.  atomic 地以新值替换</th>
    </tr>
  </table>
  <figcaption>图 6.12：在一个回圈中 atomic 递增</figcaption>
</figure>

The big differentiators are x86 and x86-64, where we have the atomic operations and, here, it is important to select the proper atomic operation to achieve the best result.
图 6.12 显示实作一个 atomic 递增操作的三种方法。在 x86 与 x86-64 上，三种方法全都会产生不同的程序，而在其它的架构上，程序则可能完全相同。效能的差异很大。下面的表格显示由四条并行的执行绪进行 1 百万次递增的执行时间。程序使用 gcc 的内建函式（`__sync_*`）

<table>
  <tr>
    <th>1. Exchange Add</th>
    <th>2. Add Fetch</th>
    <th>3. CAS</th>
  </tr>
  <tr>
    <td>0.23s</td>
    <td>0.21s</td>
    <td style="background: yellow">0.73s</td>
  </tr>
</table>

前两个数字很相近；我们看到回传旧值稍微快一点。重要的资讯在被强调的那一栏，使用 CAS 时的成本。毫不意外，它要昂贵许多。对此有诸多理由
1. 有两个memory操作;
2. CAS 操作本身比较复杂，甚至需要条件操作;
3. 整个操作必须在一个回圈中完成，以防两个同时的存取造成一次 CAS 呼叫失败。

现在读者可能会问个问题：为什么有人会使用这种利用 CAS 的复杂、而且较长的程序？对此的回答是：复杂性通常会被隐藏。如同先前提过的，CAS 是横跨所有有趣架构的统一 atomic 操作。所以有些人认为，以 CAS 定义所有的 atomic 操作就足够。这令程序更为简单。但就如数字所显示的，这绝对不是最好的结果。CAS 解法的memory管理的间接成本很大。下面示意仅有两条执行绪的执行，每条在它们自己的处理器核上。

<table>
  <tr>
    <th>执行绪 #1</th>
    <th>执行绪 #2</th>
    <th>var cache状态</th>
  </tr>
  <tr>
    <td><code>v = var</code></td>
    <td></td>
    <td>在 Proc 1 上为「E」</td>
  </tr>
  <tr>
    <td><code>n = v + 1</code></td>
    <td><code>v = var</code></td>
    <td>在 Proc 1+2 上为「S」</td>
  </tr>
  <tr>
    <td>CAS(<code>var</code>)</td>
    <td><code>n = v + 1</code></td>
    <td>在 Proc 1 上为「E」</td>
  </tr>
  <tr>
    <td></td>
    <td>CAS(<code>var</code>)</td>
    <td>在 Proc 2 上为「E」</td>
  </tr>
</table>

我们看到，在这段很短的执行期间内，cache行状态至少改变三次；两次改变为 RFO。再加上，因为第二个 CAS 会失败，所以这条执行绪必须重复整个操作。在这个操作的期间，相同的情况可能会再度发生。

相比之下，在使用 atomic 算术操作时，处理器能够将执行加法（或者其它什么的）所需的载入与储存操作保持在一起。能够确保同时发出的cache行请求直到 atomic 操作完成前都会被阻挡。

因此，在范例中的每次回圈迭代至多会产生一次 RFO cache请求，就没有别的。

这所有的一切都意味著，在一个能够使用 atomic 算术与逻辑操作的层级定义机器抽象是很重要的。CAS 不该被普遍地用作统一的机制。

对于大多数处理器而言， atomic 操作本身一直是 atomic。对于不需要 atomic 的情况，只有借由提供完全独立的程序路径时，才能够避免这点。
This means more code, a conditional, and further jumps to direct execution appropriately.

对于 x86 与 x86-64，情况就不同：相同的指令能够以 atomic 与非 atomic 的方式使用。为了令它们 atomic 化，便对指令用上一个特殊的前缀：`lock` 前缀。假如在一个给定的情况下， atomic 需求是不必要的，这为 atomic 操作提供避免高成本的机会。例如，在函式库中，在需要时必须一直是执行绪安全（thread-safe）的程序就能够受益于此。没有撰写程序时所需的资讯，决策能够在执行期进行。技巧是跳过 `lock` 前缀。这个技巧适用于 x86 与 x86-64 允许以 `lock` 前缀的所有指令。

```
    cmpl $0, multiple_threads
    je   1f
    lock
1:  add  $1, some_var
```

如果这段组合语言程序看起来很神秘，别担心，它很简单的。第一个指令检查一个变数是否为零。非零在这个情况中表示有多于一条执行中的执行绪。若是这个值为零，第二个指令就会跳到标签 `1`。否则，就执行下一个指令。这就是狡猾的部分。若是 `je` 没有跳跃，`add` 指令便会以 `lock` 前缀执行。否则，它会在没有 `lock` 前缀的情况下执行。

增加像是条件跳跃这样一个潜在昂贵的操作（在分支预测错误的情况下是很昂贵的）看似事与愿违。确实可能如此：若是大多时候都有多条执行绪在执行中，效能会进一步降低，尤其在分支预测不正确的情况。但若是有许多仅有一条执行绪在使用中的情况，程序是明显比较快的。使用一个 if-then-else 构造的替代方法在两种情况下都会引入额外的非条件跳跃，这可能更慢。假定一次 atomic 操作花费大约 200 个周期，使用这个技巧（或是 if-then-else 区块）的交叉点是相当低的。这肯定是个要记在心上的技术。不幸的是，这表示无法使用 gcc 的 `__sync_*` 内建函式。



[^40]: HP Parisc 仍然没有提供更多的操作...

[^41]: 有些人会使用「链结（linked）」而非「锁定」，这是一样的。

[^42]: x86 与 x86-64 上的 `CAS` 操作码（opcode）能够避免第二次与后续迭代中的值的载入，但在这个平台上，我们能够用一个单一的加法操作码、以一个较简单的方式来撰写 atomic 加法。

