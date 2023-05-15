# 3.3. CPU cache实作细节

cache实作者有个麻烦是，在庞大的主memory中，每个记忆单元都可能得被cache。假如一支程序的工作集足够大，这表示有许多为了cache中的各个地方打架的主memory位置。先前曾经提过，cache与主memory大小的比率为 1 比 1000 的情况并不罕见。

