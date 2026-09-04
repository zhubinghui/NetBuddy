# NetBuddy 数据流和工作流设计

**版本**: 1.0  
**更新日期**: 2026-08-21

## 1. 故障诊断工作流

### 1.1 流程概览

```
┌─────────────┐
│  故障报告   │
└──────┬──────┘
       ↓
┌──────────────────────┐
│ 初始信息收集         │
│ - 故障症状           │
│ - 受影响设备         │
│ - 故障发生时间       │
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ DiagnosisAgent 启动  │
└──────┬───────────────┘
       ↓
    [循环：逐步诊断]
   ↙  ↓  ↓  ↘
┌──┐┌──┐┌──┐┌──┐
│①LL│②CL│③TE│④DB│  ① 定位故障范围
│LL│CL│TE│DB│  ② 收集日志
└──┘└──┘└──┘└──┘  ③ 执行测试
  ↓  ↓  ↓  ↓      ④ 查询知识库
   ↖  ↓  ↓  ↙
   ┌──────────┐
   │ 根因分析 │
   └────┬─────┘
        ↓
   ┌─────────────────┐
   │ 是否需要执行变更 │
   └──┬──────────┬───┘
      是         否
      ↓         ↓
   ┌──────┐  ┌───────────┐
   │变更  │  │生成诊断报告│
   │流程  │  │并推荐方案  │
   └──────┘  └───────────┘
```

### 1.2 详细步骤

#### 第一步：问题定位

```python
# DiagnosisAgent 逻辑
async def localize_fault():
    # 1. 获取故障设备当前状态
    device_status = await device_manager.health_check(device_id)
    
    # 2. 采集关键日志
    recent_logs = await log_processor.collect_logs(
        device_id=device_id,
        time_range=(now - 1hour, now),
        log_type="error"
    )
    
    # 3. 执行基本诊断
    basic_tests = await device_manager.execute_command(
        device_id,
        "show system diagnostics"
    )
    
    # 4. 查询相似案例
    similar_cases = await knowledge_base.get_similar_cases(
        fault_symptoms
    )
    
    return {
        "device_status": device_status,
        "error_logs": recent_logs,
        "diagnostic_output": basic_tests,
        "similar_cases": similar_cases
    }
```

#### 第二步：原因分析

```python
async def analyze_root_cause(localization_result):
    # 1. 解析日志
    parsed_logs = await log_processor.parse_logs(
        localization_result["error_logs"],
        device_type
    )
    
    # 2. 提取关键信息
    key_events = extract_key_events(parsed_logs)
    
    # 3. 构建事件因果关系
    causality_graph = build_causality_graph(key_events)
    
    # 4. 应用规则引擎
    hypotheses = await knowledge_base.apply_inference_rules(
        causality_graph,
        similar_cases
    )
    
    # 5. 排序假设
    ranked_hypotheses = rank_by_confidence(hypotheses)
    
    return {
        "root_cause": ranked_hypotheses[0],
        "alternative_causes": ranked_hypotheses[1:],
        "confidence": ranked_hypotheses[0].confidence
    }
```

#### 第三步：推荐方案

```python
async def recommend_solutions(root_cause):
    # 1. 查询知识库中的相关解决方案
    solutions = await knowledge_base.search(
        root_cause.description,
        top_k=5
    )
    
    # 2. 过滤当前环境适用的方案
    applicable_solutions = filter_by_environment(
        solutions,
        current_environment
    )
    
    # 3. 评估每个方案的影响
    for solution in applicable_solutions:
        impact = await digital_twin.analyze_impact({
            "type": "solution_implementation",
            "solution_id": solution.id
        })
        solution.estimated_impact = impact
    
    # 4. 排序推荐方案
    ranked_solutions = rank_solutions(applicable_solutions)
    
    return {
        "recommended_solution": ranked_solutions[0],
        "alternative_solutions": ranked_solutions[1:],
        "requires_change": ranked_solutions[0].requires_configuration_change
    }
```

### 1.3 数据结构

```python
class DiagnosisReport:
    """诊断报告"""
    
    report_id: str
    timestamp: datetime
    
    # 故障信息
    fault_symptoms: str
    affected_devices: list[str]
    
    # 诊断过程
    localization_results: dict
    root_cause_analysis: dict
    recommended_solutions: list[Solution]
    
    # 诊断质量
    confidence_level: float  # 0-1
    diagnosis_duration: int  # 秒
    tools_used: list[str]
    
    # 后续行动
    requires_change: bool
    change_plan: Optional[ChangePlan]
    requires_manual_verification: bool
    
    # 审计信息
    diagnosed_by_agent: str
    approved_by: Optional[str]
    approval_time: Optional[datetime]
```

---

## 2. 变更操作工作流

### 2.1 流程概览

```
┌──────────────────┐
│ 变更请求提交     │
└────────┬─────────┘
         ↓
┌──────────────────────────┐
│ 变更检查和验证           │
│ - 检查权限               │
│ - 验证语法               │
│ - 检查约束               │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 模拟和影响分析           │
│ - DigitalTwin 模拟       │
│ - 计算影响范围           │
│ - 评估风险等级           │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 生成变更计划             │
│ - 逐步操作计划           │
│ - 回滚方案               │
│ - 验证步骤               │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 请求人工审批             │
│ - 主管审批               │
│ - NOC 确认               │
└────────┬─────────────────┘
         ↓ (审批通过)
┌──────────────────────────┐
│ 执行变更操作             │
│ - 按计划执行配置变更     │
│ - 实时监控               │
│ - 记录所有操作           │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 验证变更结果             │
│ - 功能测试               │
│ - 性能验证               │
│ - 告警检查               │
└────────┬─────────────────┘
         ↓
   ┌─────┴──────┐
   是否成功     否
   ↓            ↓
┌─────┐    ┌────────┐
│完成 │    │回滚    │
└─────┘    └────────┘
   ↓            ↓
┌──────────────────────┐
│ 生成变更报告         │
│ - 操作记录           │
│ - 结果总结           │
│ - 性能对比           │
└──────────────────────┘
```

### 2.2 详细步骤

#### 第一步：变更验证

```python
async def validate_change(change_request: ChangeRequest):
    """验证变更请求"""
    
    validation_results = {
        "permission_check": True,
        "syntax_check": True,
        "constraint_check": True,
        "issues": []
    }
    
    # 1. 权限检查
    if not has_permission(change_request.requester, change_request.type):
        validation_results["permission_check"] = False
        validation_results["issues"].append("权限不足")
    
    # 2. 语法检查
    if not validate_syntax(change_request.config):
        validation_results["syntax_check"] = False
        validation_results["issues"].append("配置语法错误")
    
    # 3. 约束检查
    constraint_violations = check_constraints(change_request)
    if constraint_violations:
        validation_results["constraint_check"] = False
        validation_results["issues"].extend(constraint_violations)
    
    return validation_results
```

#### 第二步：模拟和影响分析

```python
async def simulate_change(change_request: ChangeRequest):
    """模拟变更操作"""
    
    # 1. 创建隔离的仿真环境
    simulation_env = await digital_twin.create_simulation_environment(
        change_request.affected_devices
    )
    
    # 2. 在仿真环境中执行变更
    simulation_result = await digital_twin.simulate_change(
        change_request.config,
        simulation_env
    )
    
    # 3. 分析影响
    impact_analysis = {
        "affected_services": simulation_result.affected_services,
        "traffic_impact": simulation_result.traffic_impact,
        "availability_impact": simulation_result.availability_change,
        "performance_impact": simulation_result.performance_change,
        "risk_score": calculate_risk_score(simulation_result)
    }
    
    # 4. 生成回滚方案
    rollback_plan = generate_rollback_plan(
        change_request,
        simulation_result
    )
    
    return {
        "simulation_result": simulation_result,
        "impact_analysis": impact_analysis,
        "rollback_plan": rollback_plan,
        "estimated_duration": simulation_result.estimated_duration
    }
```

#### 第三步：人工审批

```python
class ApprovalRequest:
    """审批请求"""
    
    request_id: str
    change_plan: ChangePlan
    impact_analysis: dict
    risk_score: float
    
    required_approvers: list[str]  # 需要审批的角色
    approval_deadline: datetime
    
    @property
    def needs_multiple_approval(self) -> bool:
        """是否需要多人审批"""
        return self.risk_score > 50

async def request_approval(approval_request: ApprovalRequest):
    """请求审批"""
    
    # 1. 提交审批请求
    approval_id = await workflow_engine.request_approval(approval_request)
    
    # 2. 通知审批人
    await notification_service.notify_approvers(
        approval_request.required_approvers,
        f"有一个新的变更需要审批：{approval_request.change_plan.name}"
    )
    
    # 3. 等待审批
    approval_result = await workflow_engine.wait_for_approval(
        approval_id,
        timeout=approval_request.approval_deadline
    )
    
    if approval_result.approved:
        return approval_id
    else:
        raise ApprovalRejected(approval_result.rejection_reason)
```

#### 第四步：执行变更

```python
async def execute_change(change_plan: ChangePlan, rollback_plan: RollbackPlan):
    """执行变更"""
    
    execution_record = {
        "execution_id": generate_id(),
        "start_time": datetime.now(),
        "steps": [],
        "status": "in_progress"
    }
    
    try:
        # 逐步执行变更
        for step in change_plan.steps:
            step_result = await execute_change_step(step)
            
            execution_record["steps"].append({
                "step_id": step.id,
                "status": "success" if step_result.success else "failed",
                "output": step_result.output,
                "duration": step_result.duration
            })
            
            if not step_result.success and not step.ignore_failure:
                raise ChangeExecutionFailed(step.id, step_result.error)
            
            # 实时监控
            await metrics_collector.check_metrics_after_step(step)
    
    except Exception as e:
        # 如果失败，执行回滚
        await execute_rollback(rollback_plan)
        execution_record["status"] = "failed_and_rolled_back"
        execution_record["error"] = str(e)
    else:
        execution_record["status"] = "completed"
    finally:
        execution_record["end_time"] = datetime.now()
    
    return execution_record
```

#### 第五步：变更验证

```python
async def verify_change(change_plan: ChangePlan, execution_record: dict):
    """验证变更结果"""
    
    verification = {
        "functional_tests": [],
        "performance_tests": [],
        "alert_checks": [],
        "all_passed": True
    }
    
    # 1. 功能测试
    for test in change_plan.functional_tests:
        test_result = await run_functional_test(test)
        verification["functional_tests"].append(test_result)
        if not test_result.passed:
            verification["all_passed"] = False
    
    # 2. 性能验证
    before_metrics = execution_record.get("before_metrics")
    after_metrics = await metrics_collector.collect_metrics()
    
    performance_check = compare_metrics(before_metrics, after_metrics)
    verification["performance_tests"].append(performance_check)
    
    # 3. 告警检查
    alerts = await metrics_collector.get_recent_alerts()
    alert_check = {
        "unexpected_alerts": [a for a in alerts if not a.expected],
        "critical_alerts": [a for a in alerts if a.severity == "critical"]
    }
    verification["alert_checks"].append(alert_check)
    
    if alert_check["critical_alerts"]:
        verification["all_passed"] = False
    
    return verification
```

### 2.3 变更报告

```python
class ChangeReport:
    """变更报告"""
    
    change_id: str
    timestamp: datetime
    
    # 变更信息
    change_request: ChangeRequest
    change_plan: ChangePlan
    
    # 审批信息
    approval_status: str  # approved, rejected, cancelled
    approved_by: str
    approval_time: datetime
    
    # 执行信息
    execution_record: dict
    status: str  # completed, failed, rolled_back
    
    # 验证信息
    verification_result: dict
    
    # 性能对比
    metrics_before: dict
    metrics_after: dict
    performance_change: dict
    
    # 审计信息
    all_operations: list[str]  # 完整操作列表
```

---

## 3. 设备巡检工作流

### 3.1 流程概览

```
┌─────────────────┐
│ 巡检计划制定    │
└────────┬────────┘
         ↓
┌──────────────────────┐
│ 巡检准备             │
│ - 获取设备列表       │
│ - 获取基准配置       │
│ - 加载检查清单       │
└────────┬─────────────┘
         ↓
┌──────────────────────┐
│ InspectionAgent 启动 │
└────────┬─────────────┘
         ↓
  [对每个设备执行]
┌──────────────────────────┐
│ ① 连接设备               │
│ ② 执行巡检项目           │
│    - 配置巡检             │
│    - 性能巡检             │
│    - 健康巡检             │
│ ③ 采集指标和日志         │
│ ④ 对比基准值             │
│ ⑤ 识别隐患               │
└────────┬─────────────────┘
         ↓
┌──────────────────────┐
│ 聚合巡检结果         │
│ - 设备风险分级       │
│ - 整体健康评分       │
│ - 隐患汇总           │
└────────┬─────────────┘
         ↓
┌──────────────────────┐
│ 生成巡检报告         │
│ - 发现的问题列表     │
│ - 改进建议           │
│ - 优先级排序         │
└──────────────────────┘
```

### 3.2 单设备巡检检查项

| 检查项 | 方法 | 基准值 | 风险等级 |
|-------|------|-------|--------|
| **配置完整性** | 对比 | 基准配置 | Medium |
| **软件版本** | 查询版本 | 推荐版本 | High |
| **系统资源** | SNMP | 阈值 | High |
| **接口状态** | 查询状态 | 预期状态 | Medium |
| **路由完整性** | 查询路由表 | 预期路由 | High |
| **邻接关系** | 检查邻接 | 拓扑定义 | Medium |
| **日志异常** | 日志分析 | 规则库 | Medium |
| **安全检查** | 配置审计 | 安全基线 | Critical |

---

## 4. 数据流图 (DFD)

### 4.1 高层数据流

```
┌──────────────┐
│ 用户/系统    │
└──────┬───────┘
       │ 1. 事件/请求
       ↓
┌─────────────────────┐
│ API/CLI 接口        │ 2. 返回结果
├─────────────────────┤
│ - 诊断请求          ├─────────┐
│ - 变更请求          │         │
│ - 巡检计划          │         ↓
│ - 查询接口          │    ┌──────────────┐
└────────────┬────────┘    │ 用户展示     │
             │             └──────────────┘
             ↓ 3. 任务提交
       ┌──────────────┐
       │ Agent Layer  │
       └────────┬─────┘
                ↓ 4. 工具调用
        ┌───────────────────┐
        │ Tools Layer       │
        ├───────────────────┤
        │ - DeviceTools     │
        │ - DiagnosisTools  │
        │ - WorkflowTools   │
        └─────────┬─────────┘
                  ↓ 5. 请求数据
         ┌──────────────────────┐
         │ Core Services Layer  │
         └───────────┬──────────┘
                     ↓ 6. 访问存储
           ┌──────────────────┐
           │ Infrastructure   │
           ├──────────────────┤
           │ - PostgreSQL     │
           │ - Redis          │
           │ - Message Queue  │
           │ - File Storage   │
           └──────────────────┘
```

---

## 5. 时序图示例

### 5.1 故障诊断时序

```
User          API          Agent         Services       Storage
│             │             │              │              │
├─故障报告───→│             │              │              │
│             │──创建诊断───→│              │              │
│             │             │─收集日志───→│              │
│             │             │             │─查询DB────→│
│             │             │             │             │
│             │             │─模拟变更───→│              │
│             │             │             │              │
│             │             │─查询知识库──────────────→│
│             │             │             │             │
│             │             │◇根因分析     │              │
│             │             │◇推荐方案     │              │
│             │─返回结果───←│              │              │
│◄─诊断报告───│             │              │              │
```

---

**后续参考**:
- 01-architecture-overview.md - 架构总览
- 05-security-ops.md - 安全和运维设计
- 06-api-design.md - API 接口设计
