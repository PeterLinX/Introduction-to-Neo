# Neo Gas

### 介绍

Neo Gas共1亿份，代表了Neo区块链的使用权。Neo Gas会随着每个新区块的生成而产生，依照既定的缓慢衰减的发行速度，经历总量从0到1亿的过程，约22年达到1亿总量。只要获得Neo，Neo Gas便会在系统中按照算法自动生成。



### 基本概念

每一个Neo都有两种状态：unspent和spent。每一个Gas也有两种状态，available和unavailable。一个Neo的生命周期以转入地址起始，转出地址截止，转入时状态变为unspent，转出时状态变为spent。当Neo处于unspent状态时，所产生的Gas为unavailable状态，即不可提取。当Neo处于spent状态时，期间所产生的Gas变为available，用户可以提取。



### 数学模型

t_start = Neo变为unspent状态时刻

t_end = Neo变为spent状态时刻

Δt_const = t_end - t_start

Δt_var = t - t_start，t为当前时刻

可提取的Gas = f(neo_amount, Δt_const)

说明：由于$\Delta$t是定量，所以可提取的Gas也是一个定量。可提取Gas的大小取决于所持有的Neo数量以及两个状态的时间差。

不可提取的Gas = f(neo_amount, Δt_var)

说明：由于t是变量，所以不可提取的Gas也随时间增长而不停增长，是一个变量。  

  

### 用途

- 支付小蚁区块链的记账费
- 支付小蚁区块链的附加服务费
- 作为记账候选人押金



Edited by Peter Lin (https://github.com/PeterLinX/Introduction-to-Neo)
