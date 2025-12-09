# 🌙 Moonlight Butterfly - 项目指南

## 📁 项目结构

```
frontend/
├── src/
│   ├── App.vue                          # 主应用组件
│   ├── main.js                          # 应用入口
│   ├── components/                      # Vue 组件
│   │   ├── InstrumentManager.vue        # 仪器管理主组件 ✨
│   │   ├── HelloWorld.vue               # 示例组件（可删除）
│   │   ├── TheWelcome.vue               # 示例组件（可删除）
│   │   └── WelcomeItem.vue              # 示例组件（可删除）
│   └── InsManager_module/               # 仪器管理器模块
│       ├── InsManagerClinet.ts          # API 导出模块 ✨
│       ├── test.ts                      # 测试脚本
│       ├── ts_caller/                   # RPC Caller 封装
│       │   ├── main-ins-manager-caller.ts
│       │   ├── instrument-caller.ts
│       │   ├── digitizer-caller.ts
│       │   └── sync-ui-caller.ts
│       └── ts_connect/                  # WebSocket 连接层
│           ├── ts_websocket.ts
│           └── websocket-transport.ts
```

## 🚀 快速开始

### 1. 启动开发服务器

```bash
npm run dev
```

### 2. 确保 C++ 后端服务已启动

默认 WebSocket 地址: `ws://localhost:9876/websocket`

### 3. 访问应用

浏览器打开: `http://localhost:5173`

## 📝 主要功能

### 仪器管理 (InstrumentManager.vue)

- ✅ 自动连接 WebSocket
- ✅ 获取仪器列表
- ✅ 选择仪器实例
- ✅ 配置虚拟仪器
- ✅ 实时连接状态显示
- ✅ 错误提示

### API 接口 (InsManagerClinet.ts)

```typescript
// 获取仪器列表
import { getInsList } from './InsManager_module/InsManagerClinet'
const list = await getInsList()

// 获取仪器实例
import { getIns } from './InsManager_module/InsManagerClinet'
const ins = await getIns('0')

// 配置仪器
import { configInstrument } from './InsManager_module/InsManagerClinet'
const success = await configInstrument()

// 检查连接状态
import { isConnected } from './InsManager_module/InsManagerClinet'
const connected = isConnected()

// 断开连接
import { disconnect } from './InsManager_module/InsManagerClinet'
await disconnect()
```

## 🎨 组件开发

### 创建新组件

1. 在 `src/components/` 创建 `.vue` 文件
2. 导入需要的 API:

```vue
<script setup>
import { getInsList, getIns } from '../InsManager_module/InsManagerClinet'

// 你的逻辑
</script>
```

### 组件通信

使用 Vue3 Composition API:
- `ref()` - 响应式数据
- `computed()` - 计算属性
- `watch()` - 监听变化
- `onMounted()` - 生命周期钩子

## 🔧 配置

### 修改 WebSocket 地址

编辑 `src/InsManager_module/InsManagerClinet.ts`:

```typescript
const WS_URL = 'ws://your-server:port/websocket';
```

## 📦 构建生产版本

```bash
npm run build
```

构建产物在 `dist/` 目录

## 🧪 测试

### 运行测试脚本

```bash
cd src/InsManager_module
node --loader tsx test.ts
```

## 📖 扩展开发

### 添加新的 API 接口

1. 在 `InsManagerClinet.ts` 中添加新函数:

```typescript
export async function yourNewAPI(): Promise<any> {
    // 使用 mainCaller 调用底层接口
    return await mainCaller.someMethod()
}
```

2. 在组件中使用:

```vue
<script setup>
import { yourNewAPI } from '../InsManager_module/InsManagerClinet'

const result = await yourNewAPI()
</script>
```

### 添加新组件

1. 创建组件文件 `src/components/YourComponent.vue`
2. 在 `App.vue` 中导入并使用:

```vue
<script setup>
import YourComponent from './components/YourComponent.vue'
</script>

<template>
  <YourComponent />
</template>
```

## 💡 最佳实践

1. **错误处理**: 始终使用 try-catch 处理异步操作
2. **加载状态**: 使用 `loading` 状态提升用户体验
3. **响应式设计**: 确保组件在移动端也能良好显示
4. **代码复用**: 将通用逻辑提取为 composables

## 🐛 常见问题

### Q: WebSocket 连接失败？
A: 确保 C++ 后端服务已启动，检查端口 9876

### Q: 仪器列表为空？
A: 检查后端是否有配置虚拟仪器，可以点击"配置仪器"按钮

### Q: 如何调试？
A: 打开浏览器开发者工具 (F12)，查看 Console 和 Network 标签

## 📚 参考文档

- [Vue 3 文档](https://vuejs.org/)
- [Vite 文档](https://vitejs.dev/)
- [Cap'n Proto 文档](https://capnproto.org/)

## 🎯 下一步

- [ ] 添加更多仪器操作功能
- [ ] 实现数据可视化
- [ ] 添加波形显示
- [ ] 添加用户权限管理
- [ ] 优化 UI/UX

---

**Happy Coding! 🚀**

