# 🔧 仪器管理系统使用指南

## 📋 功能概览

### 1. 仪器列表管理
- ✅ 自动连接 WebSocket 服务器
- ✅ 获取可用仪器列表
- ✅ 显示仪器基本信息（名称、型号、序列号）
- ✅ 实时连接状态显示

### 2. 仪器实例操作
- ✅ 选择仪器获取实例
- ✅ 查询仪器详细信息
- ✅ 自动加载业务模块

### 3. 业务模块支持
- ✅ **SyncUI 模块** - UI 同步服务
- ✅ **Digitizer 模块** - 数字化仪服务
- ✅ 更多模块可扩展...

## 🚀 使用流程

### 步骤 1: 启动应用

```bash
npm run dev
```

### 步骤 2: 查看仪器列表

应用启动后会自动：
1. 连接到 WebSocket 服务器 (`ws://localhost:9999/websocket`)
2. 获取仪器列表
3. 显示可用仪器

### 步骤 3: 选择仪器

点击任意仪器卡片，系统会：
1. 获取仪器实例
2. 查询仪器详细信息
3. 自动加载 SyncUI 模块
4. 自动加载 Digitizer 模块

### 步骤 4: 测试模块功能

点击"测试 SyncUI"或"测试 Digitizer"按钮：
- 执行模块功能测试
- 查看控制台输出结果
- 验证模块工作状态

## 📊 界面说明

### 状态栏
显示系统状态信息：
- **连接状态**: 未连接 / 连接中... / 已连接 / 连接失败
- **仪器数量**: 当前可用仪器数量

### 仪器列表
卡片式展示所有可用仪器：
- **仪器名称**
- **仪器 ID**
- **型号**
- **序列号**
- **固件版本**（如果有）

点击卡片即可选择该仪器。

### 仪器详情

#### 基本信息
显示选中仪器的完整信息：
- 名称
- 型号
- 序列号
- 固件版本

#### 业务模块状态
实时显示模块加载状态：
- 🔄 **SyncUI**: ✅ 已加载 / ❌ 未加载
- 📊 **Digitizer**: ✅ 已加载 / ❌ 未加载

#### 快速操作
提供模块测试按钮：
- **测试 SyncUI**: 测试 UI 同步功能
- **测试 Digitizer**: 测试数字化仪功能

## 🔌 模块详解

### SyncUI 模块

**功能**：UI 数据同步服务

**测试内容**：
1. 获取 SyncUI 数据
2. 设置数据变化回调
3. 设置属性信息变化回调
4. 设置批量数据变化回调

**示例输出**：
```javascript
✅ SyncUI 数据: { componentId: '...', attributeId: 100, ... }
✅ 回调设置: 成功
📢 UI 数据变化: { ... }
```

### Digitizer 模块

**功能**：数字化仪控制服务

**测试内容**：
1. 读取通道属性值
2. 设置通道属性值
3. 查询属性范围（最大值/最小值）

**示例输出**：
```javascript
📖 读取属性...
✅ 属性值: 1.5 结果代码: 0
✍️ 设置属性...
✅ 设置结果代码: 0
🔍 查询属性范围...
✅ 范围: 0.1 ~ 10.0
```

## 🧪 开发测试

### 控制台日志

所有操作都会在浏览器控制台输出详细日志：

```javascript
🔍 选择仪器，ID: 0
✅ 获取仪器实例成功
📊 获取仪器信息...
✅ 仪器信息: { ... }
🔄 获取 SyncUI 模块...
✅ SyncUI 模块已加载
🔄 获取 Digitizer 模块...
✅ Digitizer 模块已加载
🎉 所有模块加载完成
```

### 错误处理

系统会捕获并显示错误：
- 页面顶部显示错误提示框
- 控制台输出详细错误信息
- 模块加载失败会显示警告，不会中断流程

## ⚙️ 配置

### 修改 WebSocket 地址

编辑 `src/InsManager_module/InsManagerClinet.ts`：

```typescript
const WS_URL = 'ws://your-server:port/websocket';
```

### 添加新模块

1. 在 `selectInstrument` 函数中添加模块加载逻辑
2. 在 `moduleStatus` 中添加状态
3. 在模板中添加模块卡片
4. 添加测试函数

## 📚 扩展开发

### 添加新的业务模块

```vue
<script setup>
// 1. 导入模块 Caller
import { YourModuleCaller } from '../InsManager_module/ts_caller/your-module-caller'

// 2. 添加状态
const yourModuleCaller = ref(null)
moduleStatus.value.yourModule = false

// 3. 在 selectInstrument 中加载
try {
  const client = await ins.getYourModule()
  yourModuleCaller.value = new YourModuleCaller(client)
  moduleStatus.value.yourModule = true
} catch (err) {
  console.warn('⚠️ YourModule 加载失败:', err.message)
}

// 4. 添加测试函数
async function testYourModule() {
  // 测试逻辑
}
</script>
```

### 添加自定义功能

在 `InstrumentManager.vue` 中添加新的函数和模板即可。

## 🐛 常见问题

### Q: WebSocket 连接失败？
A: 
1. 检查后端服务是否启动
2. 确认端口号是否正确（默认 9999）
3. 查看浏览器控制台的错误信息

### Q: 模块加载失败？
A: 
1. 模块加载失败不会中断流程
2. 检查服务器端是否支持该模块
3. 查看控制台的警告信息

### Q: 点击仪器无反应？
A: 
1. 检查是否有错误提示
2. 打开控制台查看详细日志
3. 确认服务器端仪器配置正确

## 🎯 下一步

- [ ] 添加波形数据显示
- [ ] 添加实时数据图表
- [ ] 添加参数配置界面
- [ ] 添加批量操作功能
- [ ] 添加数据导出功能

---

**Happy Testing! 🚀**

