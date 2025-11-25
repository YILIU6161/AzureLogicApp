# Azure Logic App - 成本监控工作流

此 Azure Logic App 工作流用于监控前一天的 Azure 成本，当成本超过 $300 时发送邮件提醒。

## 概述

工作流每天上午 9:00（中国标准时间）运行，执行以下操作：

1. **获取访问令牌**：使用服务主体凭据通过 Azure AD 进行身份验证
2. **获取成本数据**：从 Azure Cost Management API 检索前一天的成本数据
3. **解析成本响应**：从 API 响应中提取成本信息
4. **提取总成本**：计算前一天的总成本
5. **检查成本阈值**：将成本与 $300 阈值进行比较
6. **发送邮件警报**：如果成本超过 $300，通过 Office 365 发送邮件通知

## 先决条件

在部署此 Logic App 之前，请确保您拥有：

1. **Azure 订阅**：具有已启用成本管理的活动 Azure 订阅
2. **Azure AD 服务主体**：具有以下权限的服务主体：
   - 订阅上的 `CostManagementReader` 角色（或适当的成本管理权限）
   - Azure AD 中的应用程序注册
3. **Office 365 连接**：在 Logic App 中配置的 Office 365 连接器
4. **Logic App 资源**：在您的订阅中创建的 Azure Logic App 资源

## 配置

### 必需参数

在 Logic App 中配置以下参数：

1. **subscriptionId** (字符串)
   - 您的 Azure 订阅 ID
   - 示例：`12345678-1234-1234-1234-123456789012`

2. **tenantId** (字符串)
   - 您的 Azure AD 租户 ID
   - 示例：`87654321-4321-4321-4321-210987654321`

3. **clientId** (字符串)
   - 服务主体应用程序（客户端）ID
   - 在 Azure AD > 应用注册 > 您的应用 > 概述 中查找

4. **clientSecret** (字符串)
   - 服务主体客户端密钥
   - 在 Azure AD > 应用注册 > 您的应用 > 证书和密码 中创建新密钥

5. **emailRecipient** (字符串)
   - 接收成本警报的电子邮件地址
   - 示例：`admin@example.com`

6. **$connections** (对象)
   - Office 365 连接配置
   - 通过 Logic App 设计器 UI 配置此项

### 设置服务主体

1. 转到 Azure 门户 > Azure Active Directory > 应用注册
2. 点击"新注册"
3. 输入名称并点击"注册"
4. 记录 **应用程序（客户端）ID** 和 **目录（租户）ID**
5. 转到"证书和密码" > "新建客户端密码"
6. 创建密钥并**立即复制值**（之后将不再显示）
7. 转到您的订阅 > 访问控制 (IAM)
8. 点击"添加角色分配"
9. 选择角色：**成本管理读取者**
10. 分配给您的服务主体

## 部署

### 方式一：通过 Azure 门户导入

1. 打开 Azure 门户并导航到您的 Logic App
2. 在左侧菜单中点击"Logic app 设计器"
3. 在工具栏中点击"代码视图"
4. 复制 `cost-monitor-workflow.json` 的内容
5. 粘贴到代码视图编辑器中
6. 点击"保存"
7. 在 Logic App 设置中配置参数

### 方式二：通过 Azure CLI 部署

```bash
az logicapp workflow import \
  --resource-group <your-resource-group> \
  --name <your-logic-app-name> \
  --location <your-location> \
  --definition cost-monitor-workflow.json
```

### 方式三：通过 ARM 模板部署

您可以将此工作流定义包装在 ARM 模板中，以便进行自动化部署。

## 配置步骤

1. **配置 Office 365 连接**：
   - 在 Logic App 设计器中，点击"Send_Email_Alert"操作
   - 如果未连接，点击"添加新连接"
   - 使用您的 Office 365 帐户登录
   - 授权连接

2. **设置参数**：
   - 转到 Logic App > 设置 > 工作流设置
   - 或在代码视图的 `parameters` 部分配置参数
   - 更新所有必需的参数值

3. **测试工作流**：
   - 点击"运行触发器"进行手动测试
   - 检查运行历史记录以验证每个步骤是否成功执行

## 工作流详情

### 触发器

- **类型**：重复
- **频率**：每天
- **间隔**：1 天
- **计划**：上午 9:00（中国标准时间）

### 成本阈值

默认阈值设置为 **$300**。要修改：

1. 在代码视图中打开工作流
2. 找到 `Check_Cost_Threshold` 操作
3. 将值 `300` 更改为您所需的阈值
4. 相应地更新邮件正文消息

### 邮件警报内容

邮件警报包括：
- 前一天的总成本
- 阈值（$300）
- 成本期间日期
- 高重要性标志

## 监控

监控您的 Logic App 运行：

1. 转到 Logic App > 概述
2. 查看运行历史记录和执行状态
3. 点击任何运行以查看详细的执行步骤
4. 检查是否有任何错误或失败

## 故障排除

### 常见问题

1. **身份验证失败**
   - 验证服务主体凭据是否正确
   - 确保服务主体具有适当的权限
   - 检查客户端密钥是否已过期

2. **未检索到成本数据**
   - 验证订阅 ID 是否正确
   - 确保为您的订阅启用了 Cost Management API
   - 检查服务主体是否具有 `CostManagementReader` 角色

3. **未发送邮件**
   - 验证 Office 365 连接是否已配置
   - 检查电子邮件收件人地址是否有效
   - 确保 Office 365 连接器已授权

4. **未返回成本数据**
   - 成本数据可能无法立即获得
   - Azure Cost Management 数据可能有 24-48 小时的延迟
   - 如果需要，考虑调整日期范围

## 自定义

### 更改计划

修改触发器重复：

```json
"schedule": {
  "hours": [9],
  "minutes": [0]
},
"timeZone": "China Standard Time"
```

### 更改成本阈值

更新 `Check_Cost_Threshold` 操作中的阈值：

```json
"greater": [
  "@{variables('totalCost')}",
  300  // 更改此值
]
```

### 修改邮件模板

编辑 `Send_Email_Alert` 操作中的邮件正文，以自定义消息格式和内容。

## 安全最佳实践

1. **安全存储密钥**：
   - 使用 Azure Key Vault 存储客户端密钥
   - 在 Logic App 参数中从 Key Vault 引用密钥

2. **使用托管标识**（推荐）：
   - 考虑使用 Logic App 托管标识而不是服务主体
   - 减少管理客户端密钥的需要

3. **限制权限**：
   - 仅授予服务主体必要的权限
   - 使用最小权限原则

## 支持

如有问题或疑问：
- 查看 Azure Logic Apps 文档
- 查看 Azure Cost Management API 文档
- 如需要，联系 Azure 支持

## 许可证

此工作流定义按原样提供，供在 Azure Logic Apps 中使用。

