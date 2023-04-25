### 描述

该计算器是一种[**自定义操作**](https://docs.buddy.red/docs/pipelines/custom-actions)，可计算**Buddy本地部署/自托管部署**基础设施中的构建(负载)数量，并通知您是否需要添加或删除Buddy工作器。

### 配置

1. 将此存储仓分叉到您的GitHub帐户
2. 在Buddy中新建一个项目并将其与您的分叉存储仓同步
3. 该操作将自动添加到您的操作列表中，以便在整个工作区中使用。

### 输入

操作输入栏中您需要提供的信息:

- `WORKER_TAG` – 运行操作计算工作器的标签，如果您想扩展无标签的工作器，请留空。[了解标签在Buddy中的工作原理](https://docs.buddy.red/docs/on-premises/workers/workers-pipelines)
- `WORKER_SLOTS` – 每个工作器的并发插槽数，即可以同时运行多少个流水线或操作。留空以获取您帐户所设置的值。
- `MAX_WORKERS` – 在您本地部署/自托管部署中启用的最大工作器数量
- `MIN_FREE_SLOTS` – 实例中所需的空闲槽数，用于计算是否为给定标签添加或删除工作器。

### 输出

输入数据被传递给生成三个变量的`calc.sh`：

- `WORKER_TAG` – 运行操作的工作器标签
- `WORKER_SLOTS` – 每个工作器并发槽数
- `WORKERS` – 工作器优化出的数量

### 预览于Buddy GUI

![buddy-worker-calculator](https://user-images.githubusercontent.com/8556342/217527631-9c496bfa-957f-469f-8f0c-d8c81d5d7cc3.png)

### 示例流水线

在此我们可以看到一个附加了计算器操作的流水线。对于分支`main`，流水线每5分钟按计划运行一次：

```yaml
- pipeline: Worker calculator
  on: SCHEDULE
  delay: 5
  start_date: "2023-01-01T00:00:00Z"
  refs:
    - "refs/heads/main"
  actions:
    - action: Calculate Workers
      type: CUSTOM
      custom_type: Workers_Scale:latest
      inputs:
        WORKER_TAG: ""
        WORKER_SLOTS: "2"
        MAX_WORKERS: "2"
        MIN_FREE_SLOTS: "1"
```

输入描述实例中工作器的所需配置：

- 标签字段为空，说明仅指向未标记的工作器
- 实例中的每个工作器都有2个并发槽
- 最多可同时运行2个工作器
- 每个工作器上应始终有1个插槽可用

### 数据使用

该操作的输出可用于配置一个进程，该进程将自动扩展您的本地部署/自托管部署。此存储仓包含一个[example_pipeline](https://github.com/buddy-red/workers-scale/tree/main/example_pipeline)，根据可用和所需工作器的数量，它使用Terraform创建或删除工作器。可随意复制流水线并根据您的需求进行调整，或者创建您自己的扩展解决方案。
