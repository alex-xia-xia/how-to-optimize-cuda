程序性能（时间）优化

一、确定性能上限（计算访存比）
- 统计程序的输入输出数据量（忽略中间数据，得到Bytes）
- 统计程序中计算指令的op数量（以浮点数为例，FLOPs）
- 得到程序的计算访存比（以浮点数为例，FLOPs/Byte）
根据程序的计算访存比，以及部署的计算平台的性能，判定程序属于计算密集型（compute intensive），还是访存密集型（memory intensive）。也就是大家通常所说的，程序类型是compute bound还是memory bound，决定程序该如何优化，以及理论的优化极限。

1.1 计算平台指标（算力π与带宽β）
- 算力π: Maximum FLOPs Per Second

- 带宽β: Maximum Memory Access Per Second

- 计算强度上限

1.2 程序指标
- 计算量（结合实际情况计算）
- 访存量（结合实际情况计算）
提问：矩阵A+B=C，请计算一下理论时间？
- 计算强度（计算量/访存量）
1.3 Roofline模型
所谓“Roof-line”，指的就是由计算平台的算力和带宽上限这两个参数所决定的“屋顶”形态，如下图所示。
- 算力决定“屋顶”的高度（绿色线段）
- 带宽决定“房檐”的斜率（红色线段）

1.4 两个瓶颈

- 计算瓶颈
不管模型的计算强度有多大，它的理论性能最大只能等于计算平台的算力 。当模型的计算强度大于计算平台的计算强度上限时，模型在当前计算平台处于 Compute-Bound状态，即模型的理论性能受到计算平台算力的限制，无法与计算强度成正比。但这其实并不是一件坏事，因为从充分利用计算平台算力的角度上看，此时模型已经100%的利用了计算平台的全部算力。
- 带宽瓶颈
当模型的计算强度小于计算平台的计算强度上限时，由于此时模型位于“房檐”区间。因此模型理论性能的大小完全由计算平台的带宽上限（房檐的斜率）以及模型自身的计算强度所决定，此时就称模型处于 Memory-Bound 状态。
1.5 Benchmark
- 计算类型
以MatrixMul优化（或者卷积）作为模板，测试各个尺寸的最高性能。
- 带宽类型
线性内存拷贝B=A（不同的优化策略），测试各个尺寸的带宽极限。

二、Memory-Bound优化
2.1 连续访问
2.2 向量化
2.3 Cache测试Benchmark
各级cache size、各级cache的访问延迟

三、Compute-Bound优化
3.1 High-Level优化
完成上层语言级别的优化，最终生成执行代码的质量，由编译器决定。优化过程中有一些通用规则，可以参考使用。但是使用过程中，不需要过度假“优化”代码，因为编译器中端会完成常规优化（子表达式传播、常量折叠等）。High-level级别的优化，一般可以到性能极限的90%附近，具体性能视编译器而定。
- 数据复用
- 存储器优化
  1. 减少内存分配
  2. 减少内存拷贝
  3. 内存连续访问
  4. cache友好度（prefetch）
  5. 减少控制分支指令
  6. 片上可编程cache
  7. 寄存器
- 计算优化
  1. 并行粒度（与计算单元综合考虑）TLP
  2. 循环变换（分割、合并、展开、仿射、分块、重排）
  3. 向量化
  4. ILP
  5. 特殊指令
  6. 特定硬件（Tensor Core）
3.2 Low-Level优化
- 汇编反推
- 指令调度
- 指令选择
- 寄存器分配
3.3 伪编译器级别（离线调优）
- Op Fusion
- Algo Select
- CodeGen
- Deploy
3.4 体系结构
流水线冒险：1）结构冒险；2）数据冒险；3）控制冒险
其中数据冒险：1）RAW；2）WAR；3）WAW
计分板：Tomasulo算法

四、卷积常用算法
direct
im2col gemm
implicit gemm
implicit gemm splitk
winograd
fuse

五、其他优化
存储优化
启动优化
功耗优化
异构调度优化
分布式调度优化

参考文献
1. https://zhuanlan.zhihu.com/p/34204282
2. https://zhuanlan.zhihu.com/p/309318901
3. https://homes.luddy.indiana.edu/achauhan/Teaching/B629/2006-Fall/LectureNotes/27-loop-transformations.html
4. https://pages.cs.wisc.edu/~sinclair/courses/cs758/fall2019/handouts/lecture/
