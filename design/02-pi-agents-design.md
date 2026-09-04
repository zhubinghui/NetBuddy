# NetBuddy Pi Agent 框架设计

**版本**: 1.0  
**更新日期**: 2026-08-21

## 1. Pi Agent 基础概念

Pi 是一个 Agent 开发框架，核心特点：

- **工具调度**: Agent 根据任务自主决策调用哪些 Tools
- **流式执行**: 支持中途中断和控制
- **错误恢复**: 自动处理工具调用失败和重试
- **对话记忆**: 保持完整的对话历史上下文

---

## 2. NetBuddy Agent 体系

### 2.1 Agent 层级

```
┌──────────────────────────────────────────┐
│  Master Orchestrator                     │  指挥协调层
│  (任务分发、Agent 协调)                  │
├──────────────────────────────────────────┤
│  Specialized Agents                      │  专用 Agent 层
│  ├─ DiagnosisAgent       (故障诊断)     │
│  ├─ OfflineDiagAgent     (离线根因)     │
│  ├─ InspectionAgent      (设备巡检)     │
│  ├─ ChangeAgent          (变更执行)     │
│  └─ MonitorAgent         (持续监控)     │
├──────────────────────────────────────────┤
│  Tools Layer                             │  工具层
│  (DeviceTools, DiagnosisTools, etc.)     │
└──────────────────────────────────────────┘
```

### 2.2 Agent 定义模板

```python
from pi import Agent, Tool
from typing import Optional

class SpecializedAgent(Agent):
    """专用 Agent 基类"""
    
    # Agent 配置
    model = "<model-id>"  # 按部署环境配置
    system_prompt = "You are a network diagnosis expert..."
    
    # Tools 列表
    tools: list[Tool] = []
    
    # Agent 参数
    max_iterations = 50
    timeout = 300
    
    # 自定义逻辑
    def process_result(self, result):
        """处理 Agent 执行结果"""
        pass
```

---

## 3. 各类 Agent 详设

### 3.1 DiagnosisAgent (故障诊断代理)

**职责**: 在线故障诊断，实时分析

**输入参数**:
```python
{
    "fault_symptoms": str,          # 故障现象描述
    "affected_devices": list[str],  # 受影响设备列表
    "network_topology": dict,       # 网络拓扑信息
    "time_started": datetime,       # 故障发生时间
    "context": dict                 # 额外上下文信息
}
```

**执行流程**:
1. 理解故障现象
2. 采集设备状态和日志
3. 逐步缩小故障范围
4. 执行诊断测试
5. 给出根因推断和处理建议

**关键 Tools**:
- `get_device_status`: 获取设备状态
- `collect_device_logs`: 收集日志
- `run_diagnostic_test`: 运行诊断测试
- `query_knowledge_base`: 查询知识库

**输出**:
```python
{
    "status": "diagnosed" | "inconclusive",
    "root_cause": str,
    "affected_components": list[str],
    "recommendations": list[str],
    "required_actions": list[dict],
    "confidence_level": float  # 0-1
}
```

### 3.2 OfflineDiagAgent (离线诊断代理)

**职责**: 离线故障根因分析

**输入参数**:
```python
{
    "logs": list[str],              # 故障相关日志
    "config_files": dict,           # 配置文件
    "network_topology": dict,       # 网络拓扑
    "historical_incidents": list,   # 历史事件
    "device_specs": dict            # 设备规格信息
}
```

**执行流程**:
1. 解析和规范化日志
2. 识别关键事件序列
3. 构建因果关系图
4. 应用规则引擎进行推理
5. 输出根因假设和等级

**关键 Tools**:
- `parse_structured_logs`: 解析结构化日志
- `extract_event_sequence`: 提取事件序列
- `build_causality_graph`: 构建因果图
- `apply_inference_rules`: 应用推理规则

**输出**:
```python
{
    "root_cause_hypotheses": list[{
        "hypothesis": str,
        "confidence": float,
        "evidence": list[str],
        "supporting_events": list[int]
    }],
    "event_timeline": list[{
        "timestamp": datetime,
        "event": str,
        "relevance_score": float
    }],
    "recommendations": list[str]
}
```

### 3.3 InspectionAgent (巡检代理)

**职责**: 自动化设备巡检检查

**输入参数**:
```python
{
    "inspection_plan": dict,        # 巡检计划
    "devices": list[str],           # 巡检设备列表
    "check_items": list[str],       # 巡检项目
    "baseline_configs": dict        # 基准配置
}
```

**执行流程**:
1. 连接设备
2. 按计划执行各项检查
3. 采集指标和配置
4. 对比基准值
5. 生成巡检报告

**关键 Tools**:
- `connect_device`: 连接设备
- `run_inspection_check`: 执行巡检检查
- `collect_metrics`: 收集指标
- `compare_with_baseline`: 与基准对比

**输出**:
```python
{
    "inspection_results": list[{
        "device": str,
        "checks": list[{
            "check_name": str,
            "status": "pass" | "warning" | "fail",
            "value": any,
            "baseline": any,
            "message": str
        }],
        "risk_level": "low" | "medium" | "high"
    }],
    "issues_found": list[str],
    "recommendations": list[str]
}
```

### 3.4 ChangeAgent (变更代理)

**职责**: 执行网络变更操作，带风险评估

**输入参数**:
```python
{
    "change_request": dict,         # 变更请求
    "change_type": str,             # 变更类型
    "affected_devices": list[str],  # 影响设备
    "rollback_plan": dict,          # 回滚方案
    "approval_status": bool         # 审批状态
}
```

**执行流程**:
1. 验证审批状态
2. 进行变更模拟
3. 风险评估
4. 请求最终确认
5. 执行变更
6. 验证结果

**关键 Tools**:
- `simulate_change`: 模拟变更
- `assess_change_risk`: 评估风险
- `request_final_approval`: 请求最终确认
- `execute_change`: 执行变更
- `verify_change`: 验证变更结果
- `execute_rollback`: 执行回滚

**输出**:
```python
{
    "change_execution_id": str,
    "status": "success" | "partial" | "failed",
    "risk_assessment": {
        "pre_change_risk": float,
        "estimated_risk": float,
        "actual_risk": float
    },
    "changes_applied": list[{
        "device": str,
        "config_before": dict,
        "config_after": dict,
        "status": "success" | "failed"
    }],
    "rollback_needed": bool,
    "validation_results": dict
}
```

### 3.5 MonitorAgent (监控代理)

**职责**: 持续监控网络状态

**输入参数**:
```python
{
    "monitoring_plan": dict,        # 监控计划
    "alert_rules": list[dict],      # 告警规则
    "metric_thresholds": dict,      # 指标阈值
    "check_interval": int           # 检查间隔 (秒)
}
```

**执行流程** (循环):
1. 按计划采集指标
2. 评估告警条件
3. 异常检测
4. 生成告警/推送

**关键 Tools**:
- `collect_metrics`: 采集指标
- `evaluate_alert_rules`: 评估告警规则
- `detect_anomalies`: 异常检测
- `send_alert`: 发送告警

**输出**:
```python
{
    "alerts": list[{
        "alert_id": str,
        "alert_type": str,
        "severity": "critical" | "high" | "medium" | "low",
        "device": str,
        "metric": str,
        "current_value": float,
        "threshold": float,
        "timestamp": datetime,
        "description": str
    }],
    "anomalies": list[dict],
    "status_summary": dict
}
```

---

## 4. Pi Tool 设计规范

### 4.1 Tool 基础结构

```python
from pi import Tool

class NetBuddyTool(Tool):
    """NetBuddy Tool 基类"""
    
    name: str = "tool_name"
    description: str = "What this tool does"
    
    # 定义参数
    parameters: dict = {
        "param1": {
            "type": "string",
            "description": "Parameter description",
            "required": True
        }
    }
    
    # Tool 权限等级
    permission_level: str = "public"  # public, user-approved, admin-only
    
    # 执行逻辑
    async def __call__(self, **kwargs):
        """Tool 执行方法"""
        pass
    
    # 错误处理
    async def handle_error(self, error: Exception):
        """Tool 错误处理"""
        pass
```

### 4.2 Tool 权限矩阵

| Tool 类型 | 权限等级 | 需要审批 | 可中断 |
|---------|--------|--------|------|
| 信息查询 (查日志、配置) | public | ✗ | ✓ |
| 诊断测试 (ping、trace) | user-approved | ✗ | ✓ |
| 设备操作 (执行命令) | user-approved | ✓ | ✓ |
| 配置变更 | admin-only | ✓ | ✗ |
| 回滚操作 | admin-only | ✓ | ✗ |

### 4.3 Tool 错误处理

所有 Tool 应实现标准错误处理：

```python
class ToolError(Exception):
    """Tool 基础异常"""
    pass

class DeviceConnectionError(ToolError):
    """设备连接异常"""
    pass

class PermissionDeniedError(ToolError):
    """权限不足异常"""
    pass

class TimeoutError(ToolError):
    """超时异常"""
    pass
```

---

## 5. Agent 间协作模式

### 5.1 顺序协作

```
DiagnosisAgent 
    ↓ (结果)
ChangeAgent 
    ↓ (审批后)
MonitorAgent (后续监控)
```

### 5.2 并行协作

```
             OfflineDiagAgent
            ↙
DiagnosisAgent → InspectionAgent (同时执行)
            ↖
             MonitorAgent
```

### 5.3 Master Orchestrator

```python
class MasterOrchestrator:
    """Master Agent 协调器"""
    
    async def handle_incident(self, incident: dict):
        """处理事件"""
        # 1. 启动诊断
        diagnosis_result = await self.diagnosis_agent.run(incident)
        
        # 2. 如需变更，启动变更流程
        if diagnosis_result.requires_change:
            change_result = await self.change_agent.run(
                change_request=diagnosis_result.recommendations
            )
        
        # 3. 启动监控
        await self.monitor_agent.run(monitoring_config)
        
        return {
            "diagnosis": diagnosis_result,
            "change": change_result if diagnosis_result.requires_change else None,
            "monitoring": "started"
        }
```

---

## 6. 状态管理和上下文

### 6.1 Agent 状态模型

```python
class AgentState:
    """Agent 执行状态"""
    
    state_id: str               # 唯一标识
    agent_type: str             # Agent 类型
    status: str                 # running | completed | failed | cancelled
    current_task: str           # 当前任务
    progress: float             # 0-1
    context: dict               # 执行上下文
    history: list[dict]         # 执行历史
    result: dict                # 最终结果
    error: Optional[str]        # 错误信息
    start_time: datetime        # 开始时间
    end_time: Optional[datetime]# 结束时间
```

### 6.2 Agent 间上下文传递

```python
# 上游 Agent 的结果作为下游 Agent 的输入
upstream_result = {
    "agent_id": "diag_001",
    "status": "completed",
    "root_cause": "BGP session down",
    "affected_devices": ["router1", "router2"],
    "context": {
        "timestamp": "2026-08-21T10:30:00Z",
        "session_id": "sess_abc123"
    }
}

# 下游 Agent 使用此上下文
downstream_input = {
    **upstream_result,
    "change_type": "restore_bgp_session"
}
```

---

## 7. 本地 Agent 测试

### 7.1 测试框架

```python
import pytest
from pi import Agent
from netbuddy.agents import DiagnosisAgent

@pytest.mark.asyncio
async def test_diagnosis_agent():
    """测试诊断 Agent"""
    agent = DiagnosisAgent()
    
    result = await agent.run({
        "fault_symptoms": "Cannot reach 10.0.0.1",
        "affected_devices": ["router1"]
    })
    
    assert result.status == "diagnosed"
    assert result.root_cause is not None
```

### 7.2 Mock Tools

```python
# 在测试环境中 mock 设备操作
class MockDeviceConnection:
    async def execute_command(self, cmd: str):
        return "mock output"
    
    async def get_logs(self):
        return ["log line 1", "log line 2"]
```

---

## 8. 监控和日志

### 8.1 Agent 执行监控

```python
# 记录每个 Agent 执行
{
    "timestamp": "2026-08-21T10:30:00Z",
    "agent_type": "DiagnosisAgent",
    "agent_id": "diag_001",
    "status": "completed",
    "duration_seconds": 45,
    "tools_called": ["get_device_status", "collect_logs"],
    "result_summary": "diagnosis_complete",
    "user_id": "user123"
}
```

### 8.2 性能指标

- Agent 平均执行时间
- Tool 调用频率
- 诊断准确率
- 用户满意度

---

**后续参考**:
- 03-dataflow.md - 详细数据流
- 04-core-modules.md - 核心模块设计
- 05-security-ops.md - 安全和运维设计
