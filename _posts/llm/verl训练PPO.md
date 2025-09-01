---
title: verl训练PPO
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---


### 训练过程

![[ppo训练阶段.png]]

DataProto 用于封装和处理数据的核心数据结构。各阶段的数据都存放在这个数据结构中。

#### 生成阶段

从 dataloader 中取数据转成 DataProto 格式放到 batch 中。从 batch 中取出指定 key 的数据 gen_batch ，然后对 gen_batch 进行 repeat ，这个是 GRPO 的需要的，在 PPO 中不需要设置 rollout.n ，默认是 1 。对 gen_batch 进行 rollout 之后，再合并到 batch 。这个过程实际就是对 batch 的输入生成输出，然后将输入和输出合起来。为了减少数据传输的开销和内存占用，所以才先放到 gen_batch 。

#### 奖励计算

计算奖励阶段，verl 支持多源奖励机制。开启了 reward_model 就使用 reward_model 计算奖励，没有开启 reward_model 就使用 reward_fn 计算奖励。reward_model 和 reward_fn 同时生效时，reward_model 的优先级更高，先使用 reward_model 计算奖励，再使用 reward_fn 计算奖励，reward_fn 计算奖励时，如果 reward_model 的结果不为空则直接返回。reward_fn 还支持同步和异步两种方式。

#### 概率计算

基于当前批次数据 batch 计算旧策略的对数概率，从计算结果中提取策略熵，根据损失聚合模式来计算平均熵值用于训练过程的监控和分析。

如果启用了参考模型，则会计算相同输入的对数概率，以便与当前策略进行比较。默认不开启，如果开启需要配置 `algorithm.use_kl_in_reward` 或 `actor_rollout_ref.actor.use_kl_loss` 为 True 。需要注意的是，这两个参数通常不同时开启，防止过度 KL 控制。

#### 价值计算

基于当前批次数据 batch 计算状态价值。



#### 优势估计

使用广义优势估计 (GAE, Generalized Advantage Estimation) 算法，根据每个 token 的奖励值、状态价值以及响应掩码，计算出优势值 (advantages) 和回报值 (returns)。

#### Critic（价值网络）训练

通过动态绑定 `CriticWorker.update_critic()` 调用 `DataParallelPPOCritic.update_critic()` 。`DataParallelPPOCritic.update_critic()` 过程：
1. 数据预处理，从 `DataProto` 数据结构中提取关键字段的数据。
2. 将输入数据划分为多个小批量（mini-batches），每个小批量再进一步划分为微批次（micro-batches）用于训练。
3. 对于每个小批量，遍历该小批量中的所有微批次，通过 `self._forward_micro_batch(model_inputs)` 进行前向计算得到预测值 `vpreds` 。
4. 使用 `core_algos.compute_value_loss` 函数计算价值损失 `vf_loss` 和裁剪比例 `vf_clipfrac` 。
5. 对计算出的损失进行反向传播 `loss.backward()` 以累积梯度。
6. 调用 `self._optimizer_step()` 执行梯度裁剪和优化器更新，返回梯度 `grad_norm` 。

#### Actor（策略网络）训练

通过动态绑定 `ActorRolloutRefWorker.update_actor()` 调用 `DataParallelPPOActor.update_policy()` 。`DataParallelPPOActor.update_policy()` 过程：
1. 数据预处理，从 `DataProto` 数据结构中提取关键字段的数据。
2. 将输入数据划分为多个小批量（mini-batches），每个小批量再进一步划分为微批次（micro-batches）用于训练。
3. 对于每个小批量，遍历该小批量中的所有微批次，通过 `self._forward_micro_batch(model_inputs)` 进行前向计算得到熵 `entropy` 和对数概率 `log_prob` 。
4. 根据 `loss_mode` 使用对应的 `policy_loss_fn` 计算损失。
5. 将计算出的损失进行反向传播，累积梯度。
6. 调用 `self._optimizer_step()` 方法执行梯度裁剪和优化器更新，返回梯度 `grad_norm` 。


### 动态绑定

在训练过程中，很多步骤都使用了动态绑定，比如异步生成序列 `self.actor_rollout_wg.generate_sequences(gen_batch)` 、计算 reward `self.rm_wg.compute_rm_score(batch)` 等等，这样做可以灵活的通过配置来在控制实际使用哪种实现。以 `critic_output = self.critic_wg.update_critic(batch)` 为例，动态绑定的流程如下：
1. `CriticWorker` 类中的 `update_critic()` 使用 `@register` 装饰器进行注册。
2. 在 `RayWorkerGroup` 初始化的时候会调用 `_bind_worker_method()` 方法。该方法会遍历 `CriticWorker` 类中所有被 `@register` 装饰的方法，并根据装饰器中设置的属性进行处理。
3. 对于 `update_critic()` 方法， `_bind_worker_method()` 会调用 `get_predefined_dispatch_fn()` 函数，根据 `dispatch_mode` （如 `DP_COMPUTE_PROTO` ）从 `DISPATCH_MODE_FN_REGISTRY` 中获取对应的 `dispatch_fn` 和 `collect_fn` 。然后调用 `get_predefined_execute_fn()` 函数，根据 `execute_mode` （如 `Execute.ALL` ）获取对应的 `execute_fn` （如 `execute_all` ）。
4. 最后，使用 `func_generator()` 函数生成一个动态方法，并将其绑定到 `RayWorkerGroup` 实例上。



