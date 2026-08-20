# NetCare 核心模块设计

**版本**: 1.0  
**更新日期**: 2026-08-21

## 1. 模块概览

NetCare 核心服务层包含以下主要模块：

```
┌─────────────────────────────────────────────────────────┐
│              核心服务模块 (Core Modules)                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │   Device   │  │    Log     │  │  Digital   │        │
│  │  Manager   │  │  Processor │  │    Twin    │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │ Knowledge  │  │  Workflow  │  │  Metrics   │        │
│  │    Base    │  │   Engine   │  │ Collector  │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Device Manager (设备管理模块)

### 2.1 职责

- 设备连接管理
- 命令执行
- 认证和授权
- 连接池管理
- 自动重试和故障转移

### 2.2 核心组件

```python
class DeviceManager:
    """设备管理器"""
    
    def __init__(self):
        self.connection_pool = SSHConnectionPool()
        self.retry_policy = RetryPolicy()
        self.device_registry = {}
    
    # 设备注册和查询
    async def register_device(self, device_info: DeviceInfo) -> str:
        """注册设备"""
        pass
    
    async def get_device(self, device_id: str) -> Device:
        """获取设备对象"""
        pass
    
    # 连接管理
    async def connect(self, device_id: str) -> DeviceConnection:
        """建立设备连接"""
        pass
    
    async def disconnect(self, device_id: str):
        """断开设备连接"""
        pass
    
    # 命令执行
    async def execute_command(
        self,
        device_id: str,
        command: str,
        timeout: int = 30,
        sudo: bool = False
    ) -> CommandResult:
        """执行设备命令"""
        pass
    
    # 健康检查
    async def health_check(self, device_id: str) -> DeviceHealth:
        """检查设备健康状态"""
        pass
```

### 2.3 DeviceConnection 接口

```python
class DeviceConnection:
    """设备连接接口"""
    
    device_id: str
    status: str  # connected, disconnected, failed
    connected_at: datetime
    last_activity: datetime
    
    async def execute(self, command: str) -> str:
        """执行命令"""
        pass
    
    async def get_config(self) -> dict:
        """获取设备配置"""
        pass
    
    async def set_config(self, config: dict) -> bool:
        """设置设备配置"""
        pass
    
    async def is_alive(self) -> bool:
        """检查连接是否活跃"""
        pass
    
    async def close(self):
        """关闭连接"""
        pass
```

### 2.4 支持的设备类型

| 设备类型 | 连接协议 | 支持版本 |
|---------|--------|--------|
| Cisco Router | SSH | IOS, IOS XE, IOS XR |
| Cisco Switch | SSH | IOS, IOS XE |
| Juniper Router | SSH/NETCONF | Junos |
| Arista Switch | SSH, eAPI | EOS |
| Linux Server | SSH | All |
| Windows Server | WinRM, SSH | 2016+ |

### 2.5 连接池设计

```python
class SSHConnectionPool:
    """SSH 连接池"""
    
    max_connections_per_device = 5
    connection_timeout = 30
    idle_timeout = 600  # 10 分钟
    
    async def acquire(self, device_id: str) -> SSHConnection:
        """获取连接"""
        pass
    
    async def release(self, connection: SSHConnection):
        """释放连接"""
        pass
    
    async def close_idle():
        """关闭空闲连接"""
        pass
```

### 2.6 重试策略

```python
class RetryPolicy:
    """重试策略"""
    
    max_retries = 3
    initial_delay = 1  # 秒
    max_delay = 30
    backoff_factor = 2  # 指数退避
    
    retryable_errors = [
        ConnectionTimeout,
        ConnectionRefused,
        TemporaryConnectionError
    ]
```

---

## 3. Log Processor (日志处理模块)

### 3.1 职责

- 日志采集
- 日志解析和规范化
- 日志聚合和过滤
- 日志存储和查询

### 3.2 核心组件

```python
class LogProcessor:
    """日志处理器"""
    
    def __init__(self):
        self.collectors = {}
        self.parsers = {}
        self.storage = LogStorage()
    
    # 日志采集
    async def collect_logs(
        self,
        device_id: str,
        log_type: str = "all",  # all, syslog, events, debug
        time_range: tuple = None,
        limit: int = 1000
    ) -> list[LogEntry]:
        """采集日志"""
        pass
    
    # 日志解析
    async def parse_logs(
        self,
        logs: list[str],
        device_type: str
    ) -> list[ParsedLogEntry]:
        """解析日志"""
        pass
    
    # 日志聚合
    async def aggregate_logs(
        self,
        device_ids: list[str],
        time_range: tuple,
        query: dict = None
    ) -> AggregatedLogs:
        """聚合多个设备的日志"""
        pass
```

### 3.3 LogEntry 数据结构

```python
class LogEntry:
    """日志条目"""
    
    device_id: str
    timestamp: datetime
    level: str  # DEBUG, INFO, WARNING, ERROR, CRITICAL
    source: str  # syslog, event, audit, etc.
    message: str
    raw_data: str
    
    # 解析后的字段
    event_code: Optional[str]
    component: Optional[str]
    process_id: Optional[int]
    tags: list[str]
    
    @property
    def is_error(self) -> bool:
        return self.level in ["ERROR", "CRITICAL"]
    
    @property
    def is_critical(self) -> bool:
        return self.level == "CRITICAL"
```

### 3.4 日志解析器

```python
class DeviceLogParser:
    """设备日志解析器基类"""
    
    device_type: str
    supported_log_types: list[str]
    
    def parse(self, raw_log: str) -> ParsedLogEntry:
        """解析原始日志"""
        pass

# 具体实现
class CiscoIosLogParser(DeviceLogParser):
    """Cisco IOS 日志解析器"""
    pass

class JunosLogParser(DeviceLogParser):
    """Junos 日志解析器"""
    pass
```

### 3.5 日志存储

```python
class LogStorage:
    """日志存储"""
    
    # 热数据：最近 30 天
    hot_storage = PostgreSQL()
    
    # 温数据：30-90 天
    warm_storage = S3()
    
    # 冷数据：90+ 天
    cold_storage = Glacier()
    
    async def store(self, log_entry: LogEntry):
        """存储日志"""
        pass
    
    async def query(self, query: dict) -> list[LogEntry]:
        """查询日志"""
        pass
```

---

## 4. Digital Twin (数字孪生模块)

### 4.1 职责

- 网络拓扑模型维护
- 设备状态同步
- 变更模拟
- 影响分析

### 4.2 核心组件

```python
class DigitalTwin:
    """数字孪生"""
    
    def __init__(self):
        self.topology = NetworkTopology()
        self.state_manager = StateManager()
        self.simulator = ChangeSimulator()
    
    # 拓扑管理
    async def get_topology(self) -> NetworkTopology:
        """获取网络拓扑"""
        pass
    
    async def update_topology(self, topology: dict):
        """更新拓扑"""
        pass
    
    async def discover_topology(self) -> NetworkTopology:
        """自发现拓扑"""
        pass
    
    # 状态同步
    async def sync_device_state(self, device_id: str):
        """同步设备状态"""
        pass
    
    async def get_device_state(self, device_id: str) -> DeviceState:
        """获取设备状态"""
        pass
    
    # 变更模拟
    async def simulate_change(
        self,
        change_plan: ChangePlan
    ) -> SimulationResult:
        """模拟变更操作"""
        pass
    
    # 影响分析
    async def analyze_impact(
        self,
        change_plan: ChangePlan
    ) -> ImpactAnalysis:
        """分析变更影响"""
        pass
```

### 4.3 NetworkTopology 数据结构

```python
class NetworkTopology:
    """网络拓扑"""
    
    nodes: dict[str, Device]          # 设备节点
    edges: list[Link]                 # 连接关系
    hierarchies: dict[str, list[str]] # 层级关系
    vlans: dict[int, VLAN]           # VLAN 信息
    
    def get_path(self, src: str, dst: str) -> list[Link]:
        """获取源到目标的路径"""
        pass
    
    def get_neighbors(self, device_id: str) -> list[Device]:
        """获取邻接设备"""
        pass
    
    def find_affected_devices(self, device_id: str) -> list[Device]:
        """查找受影响设备"""
        pass
```

### 4.4 变更模拟

```python
class SimulationResult:
    """模拟结果"""
    
    status: str  # success, partial, failed
    changes_simulated: list[dict]
    traffic_impact: dict
    service_impact: list[str]
    risk_score: float  # 0-100
    affected_services: list[str]
    estimated_duration: int  # 秒
    rollback_possible: bool
    rollback_duration: int  # 秒
```

---

## 5. Knowledge Base (知识库模块)

### 5.1 职责

- 故障知识库维护
- 最佳实践管理
- 设备规范库
- 向量检索

### 5.2 核心组件

```python
class KnowledgeBase:
    """知识库"""
    
    def __init__(self):
        self.vector_store = VectorStore()  # 向量存储
        self.rule_engine = RuleEngine()    # 规则引擎
        self.case_library = CaseLibrary()  # 案例库
    
    # 知识检索
    async def search(
        self,
        query: str,
        top_k: int = 5
    ) -> list[KnowledgeItem]:
        """向量检索"""
        pass
    
    async def get_similar_cases(
        self,
        fault_symptoms: str
    ) -> list[FaultCase]:
        """获取相似故障案例"""
        pass
    
    # 知识存储
    async def add_fault_case(self, case: FaultCase):
        """添加故障案例"""
        pass
    
    async def add_best_practice(self, practice: BestPractice):
        """添加最佳实践"""
        pass
```

### 5.3 数据结构

```python
class FaultCase:
    """故障案例"""
    
    case_id: str
    title: str
    symptoms: str
    root_cause: str
    resolution_steps: list[str]
    devices_affected: list[str]
    resolution_time: int  # 分钟
    severity: str  # low, medium, high, critical
    keywords: list[str]
    
    # 用于向量检索
    embedding: list[float]
    
    @property
    def success_indicators(self) -> list[str]:
        """成功标志"""
        pass

class BestPractice:
    """最佳实践"""
    
    practice_id: str
    title: str
    category: str
    description: str
    steps: list[str]
    applicable_devices: list[str]
    risk_level: str
    benefit: str
```

---

## 6. Workflow Engine (工作流引擎)

### 6.1 职责

- 工作流定义和编排
- 状态管理
- 人工审批
- 审计日志

### 6.2 核心组件

```python
class WorkflowEngine:
    """工作流引擎"""
    
    def __init__(self):
        self.workflow_store = WorkflowStore()
        self.state_manager = StateManager()
        self.approval_handler = ApprovalHandler()
    
    # 工作流执行
    async def execute_workflow(
        self,
        workflow_id: str,
        input_data: dict
    ) -> WorkflowExecution:
        """执行工作流"""
        pass
    
    async def get_execution_status(
        self,
        execution_id: str
    ) -> WorkflowExecution:
        """获取执行状态"""
        pass
    
    # 审批管理
    async def request_approval(
        self,
        execution_id: str,
        approval_request: ApprovalRequest
    ) -> str:
        """请求审批"""
        pass
    
    async def approve(self, approval_id: str, approver_id: str):
        """批准"""
        pass
    
    async def reject(self, approval_id: str, approver_id: str, reason: str):
        """拒绝"""
        pass
```

### 6.3 WorkflowDefinition 格式

```yaml
# 工作流定义 (YAML)
name: "Network Change Workflow"
version: "1.0"
steps:
  - id: "step1"
    name: "验证变更"
    type: "task"
    action: "validate_change"
  
  - id: "step2"
    name: "获取审批"
    type: "approval"
    required_approvers: ["admin", "noc_lead"]
  
  - id: "step3"
    name: "执行变更"
    type: "task"
    action: "execute_change"
    depends_on: ["step2"]
  
  - id: "step4"
    name: "验证结果"
    type: "task"
    action: "verify_result"
    depends_on: ["step3"]
  
  - id: "step5"
    name: "发送报告"
    type: "task"
    action: "send_report"
    depends_on: ["step4"]
```

---

## 7. Metrics Collector (指标收集模块)

### 7.1 职责

- 性能指标采集
- 指标聚合
- 异常检测
- 告警生成

### 7.2 关键指标

| 指标 | 采集方式 | 频率 |
|-----|--------|------|
| CPU 使用率 | SNMP/CLI | 1min |
| 内存使用率 | SNMP/CLI | 1min |
| 接口速率 | SNMP | 30sec |
| 包丢失率 | ICMP | 1min |
| 延迟 | ICMP | 1min |

---

## 8. 模块间交互

### 8.1 典型流程 - 故障诊断

```
DiagnosisAgent
    ↓ 调用
DeviceManager.execute_command()
    ↓ 调用
LogProcessor.collect_logs()
    ↓ 调用
KnowledgeBase.search()
    ↓ 调用
DigitalTwin.analyze_impact()
    ↓ 返回诊断结果
DiagnosisAgent
```

### 8.2 典型流程 - 变更执行

```
ChangeAgent
    ↓ 调用
DigitalTwin.simulate_change()
    ↓ 调用
WorkflowEngine.request_approval()
    ↓ 人工审批
WorkflowEngine.execute_workflow()
    ↓ 调用
DeviceManager.execute_command()
    ↓ 调用
MetricsCollector 监控变更
```

---

**后续参考**:
- 01-architecture-overview.md - 架构总览
- 02-pi-agents-design.md - Agent 设计
- 05-security-ops.md - 安全和运维
