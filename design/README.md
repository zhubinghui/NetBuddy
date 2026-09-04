# NetBuddy 架构设计文档索引

**最后更新**: 2026-08-21  
**版本**: 1.0

## 📑 文档导航

本目录包含 NetBuddy 项目的完整架构设计文档。建议按以下顺序阅读：

### 📋 核心设计文档

#### 1. **[架构总览](01-architecture-overview.md)** — 必读 ⭐⭐⭐
   - 项目简介和愿景
   - 系统整体架构图
   - 分层设计详解
   - 关键设计原则
   - 技术栈选择
   - 开发计划
   
   **适合人员**: 所有人  
   **阅读时间**: 15-20 分钟

#### 2. **[Pi Agent 框架设计](02-pi-agents-design.md)** — 核心 ⭐⭐⭐
   - Pi Agent 基础概念
   - Agent 体系结构
   - 5 大 Agent 详细设计：
     - DiagnosisAgent (故障诊断)
     - OfflineDiagAgent (离线根因)
     - InspectionAgent (设备巡检)
     - ChangeAgent (变更执行)
     - MonitorAgent (持续监控)
   - Pi Tool 设计规范
   - Agent 间协作模式
   - 状态管理和上下文
   - 本地测试方案
   
   **适合人员**: Agent 开发者、架构师  
   **阅读时间**: 30-40 分钟

#### 3. **[核心模块设计](03-core-modules-design.md)** — 重要 ⭐⭐⭐
   - 7 大核心服务模块：
     1. **Device Manager** - 设备连接和命令执行
     2. **Log Processor** - 日志采集和分析
     3. **Digital Twin** - 网络拓扑和变更模拟
     4. **Knowledge Base** - 知识库和最佳实践
     5. **Workflow Engine** - 工作流编排
     6. **Metrics Collector** - 性能指标采集
   - 各模块详细接口设计
   - 模块间交互模式
   
   **适合人员**: 后端开发者、架构师  
   **阅读时间**: 40-50 分钟

#### 4. **[数据流和工作流](04-dataflow-workflows.md)** — 重要 ⭐⭐⭐
   - **故障诊断工作流** - 完整诊断流程
   - **变更操作工作流** - 从计划到执行的完整过程
   - **设备巡检工作流** - 自动化巡检流程
   - 数据流图 (DFD)
   - 时序图示例
   
   **适合人员**: 工作流设计、流程优化人员  
   **阅读时间**: 30-40 分钟

#### 5. **[安全和运维设计](05-security-ops.md)** — 重要 ⭐⭐
   - 安全架构（分层安全模型）
   - 身份认证和授权 (OAuth/RBAC)
   - 秘密管理 (Vault)
   - 审计和合规
   - 网络隔离和防护
   - 容灾和高可用
   - 安全测试
   - 事件响应计划
   
   **适合人员**: 安全工程师、运维工程师  
   **阅读时间**: 35-45 分钟

---

## 🎯 快速导览 (按角色)

### 👨‍💼 项目经理 / 产品经理
必读:
- [架构总览 - 项目简介和愿景](01-architecture-overview.md#1-项目简介)
- [架构总览 - 开发计划](01-architecture-overview.md#6-开发计划)
- [数据流和工作流 - 工作流概览](04-dataflow-workflows.md#1-故障诊断工作流)

### 👨‍💻 后端开发者
必读:
- [架构总览](01-architecture-overview.md) - 全部
- [核心模块设计](03-core-modules-design.md) - 重点关注你负责的模块
- [数据流和工作流 - 模块间交互](04-dataflow-workflows.md#4-数据流图-dfd)
- [安全和运维 - 基础部分](05-security-ops.md#1-安全架构)

### 🤖 AI Agent 开发者
必读:
- [架构总览 - Agent 编排层](01-architecture-overview.md#32-pi-agent-编排层)
- [Pi Agent 框架设计](02-pi-agents-design.md) - 全部
- [数据流和工作流 - 故障诊断工作流](04-dataflow-workflows.md#1-故障诊断工作流)

### 🔒 安全工程师
必读:
- [安全和运维设计](05-security-ops.md) - 全部
- [架构总览 - 关键设计原则](01-architecture-overview.md#4-关键设计原则)

### 🛠️ 运维工程师
必读:
- [架构总览 - 技术栈](01-architecture-overview.md#5-技术栈)
- [安全和运维 - 容灾和高可用](05-security-ops.md#6-容灾和高可用)
- [安全和运维 - 事件响应](05-security-ops.md#8-事件响应计划)

### 🧪 QA / 测试人员
必读:
- [数据流和工作流](04-dataflow-workflows.md) - 全部
- [Pi Agent 框架 - 本地测试](02-pi-agents-design.md#7-本地-agent-测试)

---

## 🔄 工作流快速查找

### 需要理解"从问题报告到诊断完成"?
→ 阅读 [数据流和工作流 - 故障诊断工作流](04-dataflow-workflows.md#1-故障诊断工作流)

### 需要理解"从变更请求到执行和验证"?
→ 阅读 [数据流和工作流 - 变更操作工作流](04-dataflow-workflows.md#2-变更操作工作流)

### 需要理解"如何采集和分析日志"?
→ 阅读 [核心模块设计 - Log Processor](03-core-modules-design.md#3-log-processor-日志处理模块)

### 需要理解"如何连接和管理设备"?
→ 阅读 [核心模块设计 - Device Manager](03-core-modules-design.md#2-device-manager-设备管理模块)

### 需要理解"权限系统怎么设计"?
→ 阅读 [安全和运维 - RBAC 权限管理](05-security-ops.md#22-基于角色的访问控制-rbac)

### 需要理解"Agent 之间如何协作"?
→ 阅读 [Pi Agent 框架 - Agent 间协作模式](02-pi-agents-design.md#5-agent-间协作模式)

---

## 📊 关键概念速查表

### 核心 Agents

| Agent | 职责 | 关键输入 | 关键输出 | 文档 |
|-------|------|---------|--------|------|
| **DiagnosisAgent** | 在线故障诊断 | 故障症状、设备信息 | 诊断报告、建议方案 | [02-pi-agents-design.md#31](02-pi-agents-design.md#31-diagnosisagent-故障诊断代理) |
| **OfflineDiagAgent** | 离线根因分析 | 日志、配置、拓扑 | 根因假设、优先级 | [02-pi-agents-design.md#32](02-pi-agents-design.md#32-offlinediagagent-离线诊断代理) |
| **InspectionAgent** | 设备自动巡检 | 巡检计划、设备列表 | 巡检报告、隐患列表 | [02-pi-agents-design.md#33](02-pi-agents-design.md#33-inspectionagent-巡检代理) |
| **ChangeAgent** | 变更操作执行 | 变更计划、审批状态 | 执行结果、回退方案 | [02-pi-agents-design.md#34](02-pi-agents-design.md#34-changeagent-变更代理) |
| **MonitorAgent** | 持续监控 | 监控计划、告警规则 | 告警推送、异常检测 | [02-pi-agents-design.md#35](02-pi-agents-design.md#35-monitoragent-监控代理) |

### 核心模块

| 模块 | 职责 | 关键功能 | 文档 |
|------|------|--------|------|
| **DeviceManager** | 设备连接和命令执行 | 连接池、重试、认证 | [03-core-modules.md#2](03-core-modules-design.md#2-device-manager-设备管理模块) |
| **LogProcessor** | 日志采集和分析 | 采集、解析、聚合、存储 | [03-core-modules.md#3](03-core-modules-design.md#3-log-processor-日志处理模块) |
| **DigitalTwin** | 网络拓扑和模拟 | 拓扑管理、状态同步、变更模拟 | [03-core-modules.md#4](03-core-modules-design.md#4-digital-twin-数字孪生模块) |
| **KnowledgeBase** | 知识库管理 | 向量检索、案例库、规则库 | [03-core-modules.md#5](03-core-modules-design.md#5-knowledge-base-知识库模块) |
| **WorkflowEngine** | 工作流编排 | 工作流执行、状态管理、审批 | [03-core-modules.md#6](03-core-modules-design.md#6-workflow-engine-工作流引擎) |

### 关键工作流

| 工作流 | 阶段数 | 关键决策点 | 文档 |
|-------|-------|----------|------|
| **故障诊断** | 3 阶段 | 原因确认→方案推荐 | [04-dataflow.md#1](04-dataflow-workflows.md#1-故障诊断工作流) |
| **变更执行** | 5 阶段 | 影响分析→人工审批→执行→验证→回滚 | [04-dataflow.md#2](04-dataflow-workflows.md#2-变更操作工作流) |
| **设备巡检** | 3 阶段 | 巡检准备→单设备检查→结果聚合 | [04-dataflow.md#3](04-dataflow-workflows.md#3-设备巡检工作流) |

---

## 🚀 如何使用本文档

### 第一次阅读
1. 从 [架构总览](01-architecture-overview.md) 开始
2. 根据你的角色选择相关文档深读
3. 阅读相关的工作流设计
4. 查阅具体模块的 API 设计

### 开发中参考
- 需要理解整体流程? → 查看对应的工作流文档
- 需要实现某个功能? → 查看对应的模块设计
- 需要理解权限系统? → 查看安全设计文档
- 需要新增 Agent? → 参照 Pi Agent 框架设计

### 代码实现时
- 在 Agent 实现中，参考 [02-pi-agents-design.md](02-pi-agents-design.md) 的 Tool 设计规范
- 在模块实现中，参考 [03-core-modules-design.md](03-core-modules-design.md) 的接口定义
- 在工作流实现中，参考 [04-dataflow-workflows.md](04-dataflow-workflows.md) 的流程图

---

## 📝 文档维护

### 如何建议改进
1. 在 GitHub Issues 中讨论架构相关问题
2. 提交 Pull Request 更新设计文档
3. 在 Discussion 中分享想法

### 版本历史
- **v1.0** (2026-08-21): 初始版本，包含 5 个核心设计文档

### 计划中的文档
- [ ] 06-api-design.md - REST API 详细设计
- [ ] 07-deployment.md - 部署和容器化指南
- [ ] 08-testing-strategy.md - 测试策略和覆盖
- [ ] 09-monitoring-alerting.md - 监控和告警设计
- [ ] 10-troubleshooting-guide.md - 故障排查指南

---

## 💡 设计决策记录 (ADR)

### ADR-001: 选择 Anthropic Pi 作为 Agent 框架
**状态**: 已批准  
**背景**: 需要可靠的 Agent 框架支持复杂的网络运维场景  
**决策**: 使用 Pi 框架，原因是:
- 与 Claude 模型深度集成
- 工具调度能力强大
- 错误恢复机制完善
- 支持流式执行和中断

### ADR-002: 采用 Human-in-the-loop 审批机制
**状态**: 已批准  
**背景**: 需要确保操作安全可控  
**决策**: 所有有影响的操作都需要人工审批
- 设备命令执行需审批
- 变更操作需审批
- 配置修改需审批

### ADR-003: 使用 PostgreSQL 作为主要数据库
**状态**: 已批准  
**原因**:
- 完整 ACID 支持
- 强大的 JSON 支持
- 生态完善，运维成熟

---

## 📞 获取帮助

- 🐛 **发现文档中的错误**: 提交 Issue
- 💬 **讨论架构设计**: 使用 Discussions
- 📧 **联系维护者**: 见 CONTRIBUTING.md

---

**最后修改**: 2026-08-21  
**维护者**: NetBuddy Team
