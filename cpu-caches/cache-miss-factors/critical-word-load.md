# 3.5.2. 关键word的载入

memory以比cache行大小还小的区块从主memory传输到cache中。现今是一次传输 64 *bit*，而cache行的大小为 64 或 128 *byte*。这表示每个cache行需要 8 或 16 次传输。

DRAM 晶片能够以突发（burst）模式传输那些 64 byte的区块。这能够在没有来自memory控制器的额外命令、以及可能伴随的延迟的情况下填满cache行。若是处理器预取cache行，这可能是最好的操作方式。

若是一支程序的资料或cache存取没有命中（这表示，这是个强制性cache错失〔compulsory cache miss〕–– 因为资料是第一次使用、或者是容量性cache错失〔capacity cache miss〕–– 因为受限的cache大小需要逐出cache行），情况便不同。程序继续执行所需的cache行里头的word也许不是cache行中的第一个word。即使在突发模式下、并以双倍资料速率来传输，个别的 64 bit区块也会在明显不同的时间点抵达。每个区块会在前一个区块抵达之后 4 个 CPU 周期以上抵达。若是程序继续执行所需的word是cache行的第八个，程序就必须在第一个word抵达之后，等待额外的 30 个周期以上。

事情并不必然非得如此。memory控制器能够以不同的顺序随意请求cache行的word。处理器能够传达程序正在等待哪个word –– 即*关键word*，而memory控制器能够先请求这个word。一旦这个word抵达，程序便能够在cache行其余部分抵达、并且cache还不在一致状态的期间继续执行。这个技术被称为关键word优先与提早重新启动（Critical Word First & Early Restart）。

现今的处理器实作这项技术，但有些不可能达成的情况。若是处理器预取资料，并且关键word是未知的。万一处理器在预取操作的途中请求这个cache行，就必须在不能够影响顺序的情况下，一直等到关键word抵达为止。

<figure>
  <img src="../../assets/figure-3.30.png" alt="图 3.30：在cache行末端的关键word">
  <figcaption>图 3.30：在cache行末端的关键word</figcaption>
</figure>

即使在适当的地方有了这些最佳化，关键word在cache行的位置也很重要。图 3.30 显示循序与随机存取的 Follow 测试结果。显示的是以用来巡访的指标位在第一个word来执行测试，对比指标位在最后一个word的情况下的速度减慢的结果。元素大小为 64 byte，与cache行的大小一致。数字受到许多杂讯干扰，但能够看到，一旦 L2 不再足以持有工作集大小，关键word在末端时的效能立刻就慢约 0.7%。循序存取似乎受到多一点影响。这与前面提及的、预取下个cache行时的问题一致。

