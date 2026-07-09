---
title: "test time compute"
description: "TTS、搜索、投机、概率论"
pubDatetime: 2026-07-06
tags: ["技术", "幻觉"]
draft: true
featured: false
---

先写一些关键词

TTS是什么、有什么用

TTS太长能否用投机来解决

相关文章：Claude effort level/Agentic Discovery for TTS/SVIP熵感知/Plan and Budget/AR-Sampling/Bandit Allocation/Revisiting o1 Scaling

TTS的本质

更一般地，agent能否理解为TTS

TTS 在 Agent 场景下 = **多步 tool-use 轨迹上的 Step 1/2**：

- Step 1：单条 trajectory 何时终止 tool 调用链
- Step 2：分支探索 tool 序列、利用环境 rich feedback（SDPO / RLRF）

---

如果TTS的目标是在给定算力预算B的约束下，最大化模型表现（等价的，固定延迟下最大化pass@1，固定采样次数下最大化discovery@k）

我们希望回答：

1. TTS在什么时候停下来是合适的
   1. TTS是否存在overthinking的问题（准确率平台甚至下降）
   2. 如何解决TTS的overthinking，为什么现有方法是失败的？
      1. max_token
      2. MCTS：探索多样性；但搜索空间爆炸，且难以微分
      3. PRM：引导路径向高奖励；搜索空间爆炸，容易hack
      4. RLVR：reward让模型自己学会推理；依赖coding/math等明确的reward环境
      5. SDPO/OPD：蒸馏与RL，一定程度上解决RL的reward问题
      6. **Latent Iteration和TTT：不了解**
   3. 理论层面的缺陷
      1. 没有统一的优化目标
      2. 隐空间是否存在度量？是否存在解流形？
      3. 推理路径和正确答案的关系如何在几何上表示？
   4. 一些前提
      1. TTS被认为是推理树上的随机过程——答案是路径诱导的分布，而问题在于分布何时靠近正确的解
      2. 借用混合时间的概念（将随机过程转为概率论）——假设许多题目存在performance-length曲线，且存在peak
      3. 借助信息几何的概念对上述概念进行总结——类比Muons是已知几何上的最速下降，EM的e/m-联络
   5. 具体分析过程
      1. 定义观测状态：$s_t=(h_t, entropy_t, PRM_t, fingerprint_t, …), s_{t+1}=\Phi(s_t;\theta,x)$ [可以参考最近清华大学的一篇文章]
      2. 关注弛豫/上升段、平台段、退化段
   6. 交付结果
      1. performance-length 曲线
      2. cutoff 平台段是否存在
      3. Proxy（概率论来理解随机过程） 与 peak 的对齐度
      4. 简单 adaptive stopping rule（如 PRM plateau + ‖Δh‖ < ε）
   7. 具体实验
      1. 对 AIME 子集，存在长度 t*(q)，使 pass@1在 L < t* 上升，L > t* 平台或下降
         1. 强制 CoT 长度 L ∈ {256, …, 4096}；每题 N=16 路径
         2. 记录：pass@1(L)、token entropy(L)、‖h_{t+1}−h_t‖(L)、路径聚类半径
         3. 成功：显著 peak；proxy 与 t* 相关 ρ > 0.7（可调）
         4. 模型：7B–70B 推理模型
      2. 同预算 B 下，adaptive stop（entropy / PRM / Δh）相对 fixed max_tokens，pass@1 提升超过预设阈值（如 2%），或同pass@1下 latency 显著降低
2. 能否优化现在的投机策略
