# 📋 模块支持检查机制

## 🎯 功能说明

系统会自动检查仪器支持哪些业务模块：
- ✅ 如果返回有效 Client → 初始化模块 + IAPI + SyncUI 回调
- ℹ️ 如果返回 null → 跳过该模块（仪器不支持）

## 🔍 检查流程

### 1. 获取业务模块 Client

```typescript
// InstrumentCaller 中的方法会检查返回值
const digitizerClient = await ins.getIviDigitizer()

// 返回值可能是：
// - 有效的 Client 对象 → 仪器支持此功能
// - null → 仪器不支持此功能
```

### 2. 空指针检查

**在 `instrument-caller.ts` 中**：

```typescript
async getIviDigitizer(): Promise<any> {
    const result = await this.client.getIviDigitizer().promise();
    const client = result.iviDigitizer;
    
    // 检查是否为空指针
    if (!client || client === null) {
        console.log('[InstrumentCaller] ℹ️ 仪器不支持 Digitizer 功能');
        return null;  // ✅ 返回 null 而不是抛出异常
    }
    
    return client;
}
```

**在 `InstrumentManager.vue` 中**：

```typescript
// 获取 Digitizer 模块
try {
    const digitizerClient = await ins.getIviDigitizer()
    
    // 检查是否为 null
    if (digitizerClient === null) {
        console.log('ℹ️ 仪器不支持 Digitizer 功能')
        // ✅ 跳过，不初始化模块
    } else {
        // ✅ 初始化模块
        digitizerModule.value = new DigitizerModule(digitizerClient, syncUiCaller.value)
        moduleStatus.value.digitizer = true
    }
} catch (err) {
    console.warn('⚠️ Digitizer 模块加载失败:', err.message)
}
```

### 3. 模块构造函数检查

**在业务模块构造函数中添加安全检查**：

```typescript
constructor(digitizerClient: any, syncUiCaller: SyncUiCaller) {
    // 防御性检查
    if (!digitizerClient) {
        throw new Error('Digitizer client 为空，仪器不支持此功能');
    }
    
    // 继续初始化...
}
```

## 📊 支持情况示例

### 示例 1: Digitizer 仪器

```
✅ SyncUI: 支持
✅ Digitizer: 支持（显示绿色卡片）
❌ Fgen: 不支持（返回 null，显示红色卡片）
❌ DCPwr: 不支持
❌ Transceiver: 不支持
```

### 示例 2: Fgen 仪器

```
✅ SyncUI: 支持
❌ Digitizer: 不支持
✅ Fgen: 支持（显示绿色卡片）
❌ DCPwr: 不支持
❌ Transceiver: 不支持
```

## 🎨 UI 显示

模块状态卡片会根据支持情况显示：

**支持的模块**（绿色）:
```
┌────────┐
│  📊    │
│Digitizer│
│✅已加载+ │
│  IAPI  │
└────────┘
```

**不支持的模块**（红色）:
```
┌────────┐
│  📡    │
│  Fgen  │
│❌未加载 │
└────────┘
```

## 🔧 控制台日志

### 支持的模块

```javascript
🔄 获取 Digitizer 模块...
[DigitizerModule] 🚀 初始化 Digitizer 模块...
[DigitizerModule] ✅ IAPI_Digitizer_Base 已初始化
[DigitizerModule] ✅ DigitizerCaller 已创建
[DigitizerModule] 🔄 设置 SyncUI 回调...
[DigitizerModule] ✅ SyncUI 回调设置成功
✅ Digitizer 模块已加载（含 IAPI 和回调）
```

### 不支持的模块

```javascript
🔄 获取 Fgen 模块...
[InstrumentCaller] ℹ️ 仪器不支持 Fgen 功能
ℹ️ 仪器不支持 Fgen 功能
```

或（服务器端未实现）：

```javascript
🔄 获取 Fgen 模块...
⚠️ Fgen 模块加载失败: Method not implemented
```

## 💡 最佳实践

### 1. 总是检查模块是否已加载

```vue
<template>
  <button 
    v-if="moduleStatus.digitizer" 
    @click="useDigitizer"
  >
    使用 Digitizer
  </button>
</template>

<script setup>
function useDigitizer() {
  if (!digitizerModule.value) {
    alert('Digitizer 模块未加载')
    return
  }
  
  // 安全使用模块
  const value = digitizerModule.value.getAttributeValue(1000, 0)
}
</script>
```

### 2. 优雅处理不支持的模块

```typescript
// ✅ 不要用 alert 打扰用户
// ✅ 只在控制台输出信息即可
if (digitizerClient === null) {
    console.log('ℹ️ 仪器不支持 Digitizer 功能')
    // 继续加载其他模块
}
```

### 3. 区分"不支持"和"错误"

```typescript
// 不支持（正常情况，返回 null）
if (client === null) {
    console.log('ℹ️ 仪器不支持此功能')
}

// 错误（异常情况，抛出异常）
catch (err) {
    console.warn('⚠️ 模块加载失败:', err.message)
}
```

## 🎯 效果

- ✅ 自动适配不同类型的仪器
- ✅ 只加载仪器支持的模块
- ✅ UI 实时显示模块支持状态
- ✅ 用户体验流畅，无不必要的错误提示

---

**Happy Coding! 🚀**

