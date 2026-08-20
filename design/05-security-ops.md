# NetCare 安全和运维设计

**版本**: 1.0  
**更新日期**: 2026-08-21

## 1. 安全架构

### 1.1 安全目标

- ✅ **机密性**: 保护敏感信息（密码、配置等）
- ✅ **完整性**: 确保数据和操作不被篡改
- ✅ **可用性**: 系统持续可用，防止 DoS
- ✅ **可审计性**: 所有操作都可追踪和审计
- ✅ **可控性**: 操作必须在人工授权范围内

### 1.2 分层安全模型

```
┌──────────────────────────────────────────┐
│           应用层安全                     │
│  - RBAC 权限管理                         │
│  - 审批工作流                            │
│  - 操作限流                              │
├──────────────────────────────────────────┤
│           API 层安全                     │
│  - API 认证 (JWT/OAuth)                  │
│  - API 授权 (权限检查)                   │
│  - 请求验证 (签名/加密)                  │
├──────────────────────────────────────────┤
│           传输层安全                     │
│  - TLS/SSL 加密                          │
│  - SSH 密钥管理                          │
│  - 端点验证                              │
├──────────────────────────────────────────┤
│           存储层安全                     │
│  - 密钥管理 (KMS)                        │
│  - 数据加密 (至rest)                     │
│  - 访问控制                              │
├──────────────────────────────────────────┤
│           基础设施安全                   │
│  - 网络隔离 (VPC)                        │
│  - 防火墙规则                            │
│  - 入侵检测                              │
└──────────────────────────────────────────┘
```

---

## 2. 身份认证和授权

### 2.1 认证方案

#### OAuth 2.0 / OIDC

```
┌──────────┐
│  用户    │
└────┬─────┘
     │ 1. 登录
     ↓
┌──────────────┐         ┌──────────────┐
│ NetCare      │────────→│ 身份提供商   │
│ 登录页面     │2.重定向  │ (Keycloak/   │
└──────────────┘         │  Azure AD)   │
                         └──────┬───────┘
                                │ 3. 验证身份
                                ↓
                         ┌──────────────┐
                    ┌───→│ 返回 token   │
                    │    └──────────────┘
                    │
             4. 重定向
                    │
┌──────────────────┴───┐
│ NetCare 后端         │ 5. 验证 token
│ - 交换 access token  │
│ - 获取用户信息       │
└──────────────────────┘
```

#### 实现细节

```python
# JWT Token 载荷
{
    "sub": "user123",           # 用户 ID
    "name": "John Doe",         # 用户名
    "email": "john@example.com",
    "roles": ["noc", "senior"],
    "permissions": [
        "diagnosis.execute",
        "inspection.schedule",
        "change.approve",
        "audit.view"
    ],
    "iat": 1693737600,          # 签发时间
    "exp": 1693741200,          # 过期时间
    "iss": "netcare-auth",
    "aud": "netcare-api"
}
```

### 2.2 基于角色的访问控制 (RBAC)

#### 角色定义

| 角色 | 权限 | 职责 |
|------|------|------|
| **Admin** | 全部 | 系统管理员 |
| **Engineer** | 诊断、巡检、读配置 | 网络工程师 |
| **ChangeMgr** | 变更计划、模拟 | 变更管理员 |
| **Approver** | 审批变更 | 审批人 |
| **Auditor** | 仅读审计日志 | 审计人员 |
| **Viewer** | 仅读诊断报告 | 查看人 |

#### 权限模型

```python
class Permission(Enum):
    """权限定义"""
    
    # 诊断权限
    DIAGNOSIS_VIEW = "diagnosis.view"
    DIAGNOSIS_EXECUTE = "diagnosis.execute"
    
    # 巡检权限
    INSPECTION_SCHEDULE = "inspection.schedule"
    INSPECTION_VIEW = "inspection.view"
    INSPECTION_EXECUTE = "inspection.execute"
    
    # 变更权限
    CHANGE_CREATE = "change.create"
    CHANGE_SIMULATE = "change.simulate"
    CHANGE_EXECUTE = "change.execute"
    CHANGE_APPROVE = "change.approve"
    
    # 审计权限
    AUDIT_VIEW = "audit.view"
    AUDIT_EXPORT = "audit.export"
    
    # 系统权限
    SYSTEM_ADMIN = "system.admin"

class Role(Enum):
    """角色定义"""
    
    ADMIN = "admin"
    ENGINEER = "engineer"
    CHANGE_MANAGER = "change_manager"
    APPROVER = "approver"
    AUDITOR = "auditor"
    VIEWER = "viewer"

# 角色到权限映射
ROLE_PERMISSIONS = {
    Role.ADMIN: [Permission.SYSTEM_ADMIN],  # 全部权限
    Role.ENGINEER: [
        Permission.DIAGNOSIS_VIEW,
        Permission.DIAGNOSIS_EXECUTE,
        Permission.INSPECTION_SCHEDULE,
        Permission.INSPECTION_EXECUTE,
        Permission.CHANGE_CREATE,
        Permission.CHANGE_SIMULATE,
    ],
    Role.CHANGE_MANAGER: [
        Permission.CHANGE_CREATE,
        Permission.CHANGE_SIMULATE,
        Permission.CHANGE_EXECUTE,
    ],
    Role.APPROVER: [
        Permission.CHANGE_APPROVE,
    ],
    Role.AUDITOR: [
        Permission.AUDIT_VIEW,
        Permission.AUDIT_EXPORT,
    ],
    Role.VIEWER: [
        Permission.DIAGNOSIS_VIEW,
        Permission.INSPECTION_VIEW,
        Permission.AUDIT_VIEW,
    ],
}
```

### 2.3 权限检查实现

```python
from functools import wraps

def require_permission(*permissions):
    """权限检查装饰器"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 获取当前用户和 token
            current_user = get_current_user()
            user_permissions = get_user_permissions(current_user)
            
            # 检查权限
            for required_perm in permissions:
                if required_perm not in user_permissions:
                    raise PermissionDenied(
                        f"需要权限: {required_perm}"
                    )
            
            # 执行原函数
            return await func(*args, **kwargs)
        
        return wrapper
    return decorator

# 使用示例
@router.post("/diagnosis")
@require_permission(Permission.DIAGNOSIS_EXECUTE)
async def execute_diagnosis(request: DiagnosisRequest):
    """执行诊断"""
    pass
```

---

## 3. 秘密管理

### 3.1 秘密分类

| 秘密类型 | 存储方式 | 轮换策略 | 访问控制 |
|---------|--------|--------|--------|
| **设备密码** | HashiCorp Vault | 每 90 天 | 仅 DeviceManager |
| **API 密钥** | AWS Secrets Manager | 每 180 天 | 仅相关服务 |
| **证书** | 证书管理系统 | 每 365 天 | PKI 管理 |
| **数据库密码** | Vault | 每 30 天 | 仅 DB 连接 |

### 3.2 秘密获取流程

```
┌──────────────┐
│ NetCare      │
│ 应用进程     │
└───────┬──────┘
        │ 1. 请求秘密 (secret_id)
        ↓
┌───────────────────┐
│ Vault Agent       │
│ (sidecar)         │
└───────┬───────────┘
        │ 2. 转发请求
        ↓
┌───────────────────┐
│ Vault 服务器      │
│ (验证 identity)   │
└───────┬───────────┘
        │ 3. 返回秘密
        ↓
┌───────────────────┐
│ 应用使用秘密      │
└───────────────────┘
```

### 3.3 实现示例

```python
from hvac import Client as VaultClient

class SecretManager:
    """秘密管理器"""
    
    def __init__(self):
        self.vault_client = VaultClient(
            url="https://vault.netcare.internal:8200"
        )
    
    async def get_device_credentials(self, device_id: str) -> dict:
        """获取设备凭证"""
        
        # 从 Vault 获取秘密
        secret = self.vault_client.secrets.kv.v2.read_secret_version(
            path=f"devices/{device_id}/credentials"
        )
        
        return {
            "username": secret["data"]["data"]["username"],
            "password": secret["data"]["data"]["password"],
            # 不要返回原始密码，使用临时令牌
        }
    
    async def rotate_secret(self, secret_path: str):
        """轮换秘密"""
        
        # 1. 生成新秘密
        new_secret = generate_random_password()
        
        # 2. 更新设备密码
        # ... (通过设备连接更新)
        
        # 3. 更新 Vault
        self.vault_client.secrets.kv.v2.create_or_update_secret(
            path=secret_path,
            secret_dict={"password": new_secret}
        )
        
        # 4. 记录轮换事件
        await audit_logger.log_secret_rotation(secret_path)
```

---

## 4. 审计和合规

### 4.1 审计日志架构

```python
class AuditLog:
    """审计日志"""
    
    log_id: str
    timestamp: datetime
    
    # 用户信息
    user_id: str
    user_email: str
    user_ip: str
    
    # 操作信息
    action: str  # diagnosis, change, approval, etc.
    resource: str  # 受影响的资源
    resource_id: str
    
    # 操作详情
    operation_type: str  # create, read, update, delete, execute
    status: str  # success, failure
    details: dict  # 操作详情
    
    # 结果
    result_summary: str
    error_message: Optional[str]
    
    # 是否敏感操作
    is_sensitive: bool
    required_approval: bool
    
    # 变更前后
    before_state: Optional[dict]
    after_state: Optional[dict]
```

### 4.2 审计日志记录点

| 操作 | 记录内容 | 敏感度 |
|-----|--------|-------|
| 用户登录 | 用户 ID、时间、IP | Medium |
| 权限变更 | 用户、新权限、旧权限 | High |
| 诊断执行 | Agent、设备、症状 | Low |
| 变更批准 | 批准人、变更 ID、决定 | High |
| 变更执行 | 执行者、变更内容、结果 | Critical |
| 配置修改 | 修改前后、设备、用户 | Critical |
| 秘密访问 | 用户、秘密 ID、时间 | Critical |

### 4.3 实现示例

```python
class AuditLogger:
    """审计日志记录器"""
    
    def __init__(self):
        self.log_storage = PostgreSQL()
        self.log_queue = MessageQueue()
    
    async def log_action(self, audit_log: AuditLog):
        """记录审计日志"""
        
        # 1. 立即存储到数据库
        await self.log_storage.insert_audit_log(audit_log)
        
        # 2. 发送到消息队列（异步处理）
        await self.log_queue.publish(
            topic="audit_logs",
            message=audit_log.model_dump_json()
        )
        
        # 3. 如果是敏感操作，立即告警
        if audit_log.is_sensitive:
            await self.alert_on_sensitive_operation(audit_log)
    
    async def log_change_execution(
        self,
        change_id: str,
        user_id: str,
        before_config: dict,
        after_config: dict,
        status: str
    ):
        """记录变更执行"""
        
        audit_log = AuditLog(
            log_id=generate_id(),
            timestamp=datetime.now(),
            user_id=user_id,
            action="change_execution",
            operation_type="update",
            status=status,
            is_sensitive=True,
            required_approval=True,
            before_state=before_config,
            after_state=after_config
        )
        
        await self.log_action(audit_log)

# 使用示例
audit_logger = AuditLogger()

async def execute_change(change_plan: ChangePlan):
    """执行变更"""
    
    # 记录变更执行
    await audit_logger.log_change_execution(
        change_id=change_plan.id,
        user_id=current_user.id,
        before_config=get_current_config(),
        after_config=change_plan.target_config,
        status="success"
    )
```

### 4.4 合规支持

- ✅ **SOC 2 Type II**: 完整审计日志、访问控制
- ✅ **ISO 27001**: 信息安全管理
- ✅ **GDPR**: 用户数据保护、删除权
- ✅ **HIPAA**: 用于医疗网络环境

---

## 5. 网络隔离和防护

### 5.1 网络架构

```
┌──────────────────────────────────────┐
│           外部网络                    │
│        (互联网/公网)                  │
└──────────────────┬───────────────────┘
                   │
            ┌──────▼─────┐
            │   防火墙   │
            │  (入站规则)│
            └──────┬─────┘
                   │
    ┌──────────────┴──────────────┐
    │    DMZ 区域                  │
    │  ┌─────────────────────┐    │
    │  │ API Gateway         │    │
    │  │ (TLS 终止点)        │    │
    │  └──────────┬──────────┘    │
    └─────────────┼────────────────┘
                  │ 内网通信
    ┌─────────────┴──────────────┐
    │   私有网络 (VPC)            │
    │  ┌─────────────────────┐   │
    │  │ NetCare 应用集群    │   │
    │  │ - API Server        │   │
    │  │ - Agent 服务        │   │
    │  │ - Worker            │   │
    │  └─────────────────────┘   │
    │  ┌─────────────────────┐   │
    │  │ 数据存储            │   │
    │  │ - PostgreSQL        │   │
    │  │ - Redis             │   │
    │  │ - S3                │   │
    │  └─────────────────────┘   │
    └─────────────────────────────┘
                  │
    ┌─────────────┴──────────────┐
    │   设备网络区域              │
    │  ┌─────────────────────┐   │
    │  │ 网络设备            │   │
    │  │ - Router            │   │
    │  │ - Switch            │   │
    │  │ - Firewall          │   │
    │  └─────────────────────┘   │
    └─────────────────────────────┘
```

### 5.2 防火墙规则

```yaml
# 入站规则
inbound_rules:
  - from: external
    port: 443
    protocol: tcp
    action: allow
    description: "HTTPS API 访问"
  
  - from: external
    port: 22
    protocol: tcp
    action: allow_with_vpn
    description: "SSH 访问 (需 VPN)"

# 出站规则
outbound_rules:
  - to: netcare_subnet
    port: all
    protocol: tcp/udp
    action: allow
    description: "到 NetCare 网络"
  
  - to: device_network
    port: [22, 23, 161, 443]
    protocol: tcp/udp
    action: allow
    description: "到设备网络"
  
  - to: external
    port: 443
    protocol: tcp
    action: allow
    description: "出站 HTTPS (更新、API)"
```

---

## 6. 容灾和高可用

### 6.1 高可用架构

```
┌──────────────────────────────────────┐
│      AWS / 云平台                    │
├──────────────────────────────────────┤
│                                      │
│  ┌──────────────┐  ┌──────────────┐ │
│  │  可用区 1    │  │  可用区 2    │ │
│  │  ┌────────┐  │  │  ┌────────┐  │ │
│  │  │ App x3 │  │  │  │ App x3 │  │ │
│  │  └────────┘  │  │  └────────┘  │ │
│  │  ┌────────┐  │  │  ┌────────┐  │ │
│  │  │ Worker │  │  │  │ Worker │  │ │
│  │  │ Pool   │  │  │  │ Pool   │  │ │
│  │  └────────┘  │  │  └────────┘  │ │
│  └──────────────┘  └──────────────┘ │
│        ↓                    ↓        │
│  ┌─────────────────────────────┐   │
│  │  跨 AZ 负载均衡              │   │
│  │  (Route 53 / ALB)            │   │
│  └─────────────────────────────┘   │
│                                      │
│  ┌──────────────┐  ┌──────────────┐ │
│  │ PostgreSQL   │  │  Redis       │ │
│  │ Primary      │  │  Cluster     │ │
│  │ (AZ1)        │  │              │ │
│  └────────┬─────┘  └──────────────┘ │
│           │ replication               │
│  ┌────────▼─────┐                   │
│  │ PostgreSQL   │                   │
│  │ Standby      │                   │
│  │ (AZ2)        │                   │
│  └──────────────┘                   │
│                                      │
└──────────────────────────────────────┘
```

### 6.2 可用性指标

| 指标 | 目标 | 实现方式 |
|-----|------|--------|
| **API 可用性** | 99.9% | 多 AZ 部署、LB、自动故障转移 |
| **数据库可用性** | 99.99% | 主从复制、自动转移 |
| **Agent 可用性** | 99% | 多实例、队列备用 |
| **RTO** | < 5 分钟 | 自动转移 |
| **RPO** | < 1 分钟 | 同步复制 |

### 6.3 灾难恢复计划

```
灾难类型          触发条件           恢复步骤
─────────────────────────────────────
数据中心故障      所有 AZ 不可用      1. 激活 DR 站点
                                      2. 重新指向 DNS
                                      3. 恢复服务
                                      
数据库故障        主库无响应          1. 自动故障转移到备库
                  > 30 秒             2. 重新创建新备库
                                      3. 验证数据完整性
                                      
应用崩溃          多个 pod 离线        1. K8s 自动重启
                                      2. 通知告警
                                      3. 人工介入
                                      
外部 API 故障     依赖服务不可用      1. 启用本地缓存
                                      2. 队列请求
                                      3. 降级服务
```

---

## 7. 安全测试

### 7.1 测试类型

| 测试类型 | 频率 | 范围 |
|---------|------|------|
| **SAST** | 每次代码提交 | 代码 |
| **DAST** | 每周 | 运行中的应用 |
| **IAST** | 每周 | 端到端 |
| **渗透测试** | 每季度 | 全系统 |
| **依赖扫描** | 每天 | 所有依赖 |

### 7.2 安全测试清单

- [ ] 权限提升攻击测试
- [ ] SQL 注入测试
- [ ] XSS 攻击测试
- [ ] CSRF 攻击测试
- [ ] 认证绕过测试
- [ ] 秘密泄露扫描
- [ ] 依赖漏洞扫描
- [ ] 配置错误检查

---

## 8. 事件响应计划

### 8.1 安全事件分类

| 级别 | 定义 | 响应时间 | 升级条件 |
|------|------|--------|--------|
| **Critical** | 数据泄露、系统被控制 | < 15 分钟 | 立即 |
| **High** | 权限被提升、异常变更 | < 1 小时 | > 5 分钟无响应 |
| **Medium** | 多次认证失败、异常访问 | < 4 小时 | > 1 小时无响应 |
| **Low** | 警告告警、配置偏差 | < 1 天 | > 4 小时无响应 |

### 8.2 响应流程

```
┌──────────────────┐
│ 检测到安全事件   │
└────────┬─────────┘
         ↓
┌──────────────────────────┐
│ 事件分类和评估           │
│ - 确定严重级别           │
│ - 确定影响范围           │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 激活应急响应             │
│ - 召集应急小组           │
│ - 隔离受影响系统         │
│ - 启动通信协议           │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 调查和分析               │
│ - 收集取证信息           │
│ - 分析攻击向量           │
│ - 评估数据泄露           │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 遏制和恢复               │
│ - 切断攻击来源           │
│ - 恢复受影响系统         │
│ - 验证系统完整性         │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ 事后审计                 │
│ - 编写事件报告           │
│ - 改进防御措施           │
│ - 更新安全策略           │
└──────────────────────────┘
```

---

**后续参考**:
- 01-architecture-overview.md - 架构总览
- 03-core-modules-design.md - 核心模块设计
- 06-api-design.md - API 接口设计（待编写）
