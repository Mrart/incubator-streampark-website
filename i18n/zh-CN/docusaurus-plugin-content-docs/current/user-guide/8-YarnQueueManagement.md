---
id: 'YarnQueueManagement'
title: 'Yarn队列管理'
sidebar_position: 8
---

## 背景介绍

在实际的生产环境中，针对 `Yarn 部署模式`，
用户通常需要输入 `queue` 或 `queue`&`labels` 来指定
`yarn-application` 模式下的 Flink 应用程序
或 `yarn-session` 模式下的 Flink 集群的配置。
在此过程中，用户的手动输入可能会导致错误，
可能会导致指定不存在的队列，或者将 `flink 应用程序` / `flink 集群` 
提交到错误的队列中。

如果 Yarn 集群中不包含用户指定的队列，
那么部署 `flink 应用程序` / `flink 集群` 的过程
将会耗时，并伴随着糟糕的用户体验。
如果由于输入错误而将任务提交到错误的队列中，
可能会影响队列上 Yarn 应用程序的稳定性，并滥用队列资源。

因此，StreamPark 引入了队列管理功能，以确保一组添加的队列在同一团队内共享，
也就是确保队列资源在团队范围内是隔离的。它可以产生以下好处：
- 当部署 Flink `yarn-application` 应用程序或 Flink `yarn-session` 集群时，
它可以快速准确地设置 Yarn 队列（`yarn.application.queue`）和标签（`yarn.application.node-label`）。
- 它不仅确保了队列和标签输入的正确性，
而且缩短了由于错误队列导致应用程序失败的时间消耗，
即尽早提示用户队列设置结果是否正确，提高交互效率。
- 有效防止滥用队列资源。

## 如何创建Yarn队列
- 前提条件
  您必须保留一个管理员用户帐户。在本例中，管理员用户 `admin` 用于登录系统中的 `default` 团队。
- 定位 `Yarn 队列` 页面的 `添加` 按钮，按照以下步骤和图示操作：
  1. 使用管理员用户帐户登录系统，并选择目标团队。
  2. 单击 `设置`。
  3. 单击 `Yarn 队列`。
  4. 单击 `添加`。
  <img src="/doc/image/yarn-queue/flow_to_create.png"/><br></br><br></br>

- 在队列字段中输入信息。
  - 创建一个干净的队列。
    - 输入所需的yarn队列名称
    - 如果需要，输入可选的yarn队列描述。
    - 单击`确认`。
    <img src="/doc/image/yarn-queue/flow_to_type_in_pure_queue.png"/><br></br><br></br>
  - 创建一个带有标签的队列。
    - 输入所需的yarn队列标签名称。
      队列标签，例如`queue1`表示队列名称，`queue@label1，label2`表示队列名称为`queue1`，队列的标签为`label1`和`label2`。
    - 如有需要，输入yarn队列的可选描述。
    - 点击`确认`。
  <img src="/doc/image/yarn-queue/flow_to_type_in_pure_queue_labels.png"/><br></br><br></br>
- 查看已存在的yarn队列(及标签)。
  <img src="/doc/image/yarn-queue/existed_queues.png"/><br></br><br></br>


## 如何使用可用队列

- 创建一个指定的yarn队列的Flink集群。
  
  <img src="/doc/image/yarn-queue/available_queues_when_creating_cluster.png"/><br></br><br></br>

- 创建一个指定 YARN 队列的 YARN 应用程序模式下的 Flink 应用程序。

  <img src="/doc/image/yarn-queue/use_yarn_app_mode_to_create_application.png"/><br></br><br></br>
  <img src="/doc/image/yarn-queue/available_queues_when_creating_application.png"/><br></br><br></br>


## 该功能的相关条目

- 这个特性会影响使用旧动态属性指定了`yarn-application`模式下的flink应用程序和`yarn-session`模式下的flink集群的yarn队列(`yarn.application.queue`)和标签(`yarn.application.node-label`)的应用程序。  
> 该功能不会对其产生任何影响。StreamPark仍然保留动态属性的最高优先级，并不强制验证动态属性中指定的队列和标签，为用户提供高级配置的空间。

- Yarn队列的隔离并不严格。
> 在当前的设计中，由于队列与`yarn-session`模式下的Flink集群之间的关系，队列的权限并没有被严格隔离。
> Flink的`yarn-session`集群对所有团队可见，如下图所示。目标`yarn-session` Flink集群在`default`团队中使用一个队列，但它仍然可以在`test`团队中选择。
> 因此，如果`yarn-session` Flink集群使用位于某个团队的队列，随着目标集群的资源使用量增加，目标资源使用的队列资源也会增加。这就是由于队列和`yarn-session`模式Flink集群之间的关系导致队列无法被严格隔离的现象。
> 换句话说，在`yarn-session`模式下，Flink集群的共享导致团队之间间接共享队列资源。

  <img src="/doc/image/yarn-queue/use_yarn_session_mode_to_create_application.png"/><br></br>

- 会话集群被所有团队共享。为什么创建 `yarn-session` Flink 集群时，只能使用当前团队中的队列而非所有团队中的队列作为候选队列列表？
> 基于上述所提到的，StreamPark 希望在创建 `yarn-session` Flink 集群时，管理员只能指定当前团队所属的队列，这有助于管理员更好地感知当前操作对当前团队的影响。

- 为什么不支持将 `flink yarn-session clusters / general clusters` 在团队范围内进行隔离？
  - 集群可见性的变化带来的影响范围比队列可见性的变化范围更大。
  - StreamPark 需要面对更大的向后兼容性难题，同时还需要考虑用户体验。
  - 目前，社区对使用 `yarn-application` 和 `yarn-session` 集群模式部署的用户群体和应用规模没有确切的研究。
    基于这一事实，社区没有提供更大的功能支持。

- 如果您对该功能有任何相关需求，请随时与我们联系或直接向社区邮件列表提供反馈。社区将根据用户组和应用程序规模的使用情况进行下一步评估和支持。

欢迎提出任何建议。
