# AXI DMA详解

## 1. AXI interfaces in Zynq



方式 | 优点 | 缺点 | 建议用途 | 估计吞吐量
-|-|-|-|-|-
CPU控制的IO | - 软件简单<br>- 最少的逻辑资源<br>- 逻辑接口简单 | 吞吐率最低 | 控制 | <25 MB/s
PS的DMAC | - 最少的逻辑资源<br>- 吞吐率中等<br>- 多通道<br>- 逻辑接口简单 | - DMAC配置有一定难度 | 当PL的DMA不够时 | 600MB/s
PL的DMA和AXI_HP | - 吞吐率最高<br>- 多个接口<br>- 有FIFO缓存 | - 只能访问OCM和DDR<br>- 设计复杂 | - 大块数据高性能传输 | 1200 MB/s（每个接口）
PL的DMA和AXI_ACP | - 吞吐率最高<br>- 延时最低<br>- 可选的Cache<br>一致性 | - 大块数据传输引起Cache问题<br>共享了CPU的互联带宽<br>更复杂的逻辑设计 | 小块的与Cache直接相关的<br>高速传输 | 1200 MB/s（每个接口）
PL的DMA和AXI_GP | 吞吐率中等 | 更复杂的逻辑设计 | PL到PS的控制功能<br>PS I/O外设访问 | 600 MB/s（每个接口）


## 1. Parameters for a transaction by DMA controller

- Source address
- Destination address
- Length of data

