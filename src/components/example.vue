<template>
    <!-- Toast 提示 -->
    <Transition name="toast">
        <div v-if="toast.show" class="toast" :class="toast.type">
            <span class="toast-icon">
                {{ toast.type === 'success' ? '✅' : toast.type === 'error' ? '❌' : '⚠️' }}
            </span>
            <span class="toast-message">{{ toast.message }}</span>
        </div>
    </Transition>
    
    <div class="container">
        <div class="left-panel" :style="{ flexBasis: leftWidth + '%' }">
            <h2>仪器列表</h2>
            <div v-if="loading && instrumentList.length === 0" class="loading-state">
                <span class="spinner"></span>
                <span>正在加载仪器列表...</span>
            </div>
            <div v-else class="instrument-list">
                <Insument
                    v-for="ins in instrumentList"
                    :key="ins.id"
                    :name="ins.name || ins.id"
                    :model="ins.model || '未知型号'"
                    :ins-type="ins.InsType || ''"
                    :address="ins.address || ins.visa || ''"
                    :status="ins.instance ? '可用' : '不可用'"
                    :active="selectedIns?.id === ins.id"
                    :instance="ins.instance"
                    @click="selectInstrument(ins)"
                />
            </div>
        </div>
        
        <!-- 可拖动分隔条 -->
        <div class="resizer" @mousedown="startResize"></div>
        
        <div class="right-panel">
            <div v-if="selectedIns" class="instrument-detail">
                <h1>{{ selectedIns.name || selectedIns.id }}</h1>
                
                <!-- 加载状态 -->
                <div v-if="loading" class="loading-state">
                    <span class="spinner"></span>
                    <span>正在获取仪器实例...</span>
                </div>
                
                <!-- 基本信息（来自列表） -->
                <div class="detail-section">
                    <h3>📋 基本信息</h3>
                    <div class="detail-item">
                        <span class="label">型号：</span>
                        <span class="value">{{ selectedIns.model || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">类别：</span>
                        <span class="value">{{ selectedIns.InsType || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">地址：</span>
                        <span class="value mono">{{ selectedIns.address || selectedIns.visa || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">状态：</span>
                        <span class="value" :class="{ available: selectedIns.connected }">
                            {{ selectedIns.status || (selectedIns.connected ? '可用' : '不可用') }}
                        </span>
                    </div>
                </div>
                
                <!-- 详细信息（来自仪器实例） -->
                <div v-if="insInfo && !loading" class="detail-section">
                    <h3>🔧 仪器实例信息</h3>
                    <div class="detail-item">
                        <span class="label">厂商：</span>
                        <span class="value">{{ insInfo.manufacturer || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">型号：</span>
                        <span class="value">{{ insInfo.model || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">序列号：</span>
                        <span class="value mono">{{ insInfo.serialNumber || '未知' }}</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">固件版本：</span>
                        <span class="value">{{ insInfo.firmwareVersion || '未知' }}</span>
                    </div>
                </div>
                
                <!-- 属性列表（动态从 attributeMap 生成） -->
                <div v-if="selectedIns?.instance?.iviClassWrapper?.attributeMap" class="detail-section attr-section">
                    <h3>🎛️ {{ getInsTypeIcon(selectedIns.instance.insType) }} {{ selectedIns.instance.insType }} 属性</h3>
                    <div class="attr-grid">
                        <div 
                            v-for="[attrId, attr] in selectedIns.instance.iviClassWrapper.attributeMap"
                            :key="attrId"
                            class="attr-card"
                        >
                            <div class="attr-header">
                                <span class="attr-id">ID: {{ attrId }}</span>
                                <span class="attr-name">{{ attr.attr_name }}</span>
                                <span class="attr-type">{{ attr.ValueType }}</span>
                            </div>
                            
                            <!-- 范围信息 -->
                            <div class="attr-range">
                                <template v-if="attr.rangeType === 0">
                                    <!-- CONTINUOUS 连续型 -->
                                    <span class="range-item">
                                        <span class="range-label">范围:</span>
                                        <span class="range-value">{{ attr.min }} ~ {{ attr.max }}</span>
                                    </span>
                                    <span class="range-item" v-if="attr.step">
                                        <span class="range-label">步进:</span>
                                        <span class="range-value">{{ attr.step }}</span>
                                    </span>
                                </template>
                                <template v-else-if="attr.rangeType === 1">
                                    <!-- OPTIONS 选项型 -->
                                    <span class="range-item options">
                                        <span class="range-label">选项:</span>
                                        <span class="range-value">{{ attr.options?.join(' | ') }}</span>
                                    </span>
                                </template>
                            </div>
                            
                            <div class="attr-body">
                                <div class="attr-display">
                                    <span class="label">当前值</span>
                                    <span class="value">{{ attr.value}}</span>
                                    <button class="btn-get" @click="onAttrClick(attr, '-1')">获取</button>
                                </div>
                                <div class="attr-set">
                                    <span class="label">设置值</span>
                                    <input 
                                        type="text" 
                                        :value="setValues[attr.attr_id] ?? attr.value"
                                        @input="setValues[attr.attr_id] = $event.target.value"
                                        class="attr-input"
                                        placeholder="输入新值"
                                        @click.stop
                                    />
                                    <button class="btn-set" @click="onAttrSet(attr, '-1')">设置</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- 仪器实例状态 -->
                <div v-if="selectedIns?.instance && !loading" class="detail-section instance-status">
                    <h3>✅ 仪器实例已获取</h3>
                    <p class="hint">属性值会通过 SyncUI 自动同步更新（响应式）</p>
                </div>
            </div>
            <div v-else class="empty-state">
                <p>请从左侧选择一个仪器</p>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { MainInsManagerCaller } from '../InsManager_module/ts_caller/index'
import Insument from './insument.vue'

// 服务器地址
const WS_URL = 'ws://localhost:9999/websocket'

// 创建 MainInsManagerCaller 实例
const mainCaller = new MainInsManagerCaller(WS_URL)

const instrumentList = ref([])  // 包含 instance 的仪器列表
const selectedIns = ref(null)
const insInfo = ref(null)  // 选中仪器的详细信息
const loading = ref(false)
const setValues = ref({})  // 存储每个属性的设置值 { attr_id: value }

// 面板宽度调整
const leftWidth = ref(20)  // 左侧面板初始宽度百分比
const isResizing = ref(false)

// Toast 提示
const toast = ref({
    show: false,
    message: '',
    type: 'success'  // 'success' | 'error' | 'warning'
})

const showToast = (message, type = 'success', duration = 2000) => {
    toast.value = { show: true, message, type }
    setTimeout(() => {
        toast.value.show = false
    }, duration)
}

// 面板宽度调整函数
const startResize = (e) => {
    isResizing.value = true
    document.addEventListener('mousemove', handleResize)
    document.addEventListener('mouseup', stopResize)
    document.body.style.cursor = 'col-resize'
    document.body.style.userSelect = 'none'
}

const handleResize = (e) => {
    if (!isResizing.value) return
    
    const containerWidth = window.innerWidth
    const newWidth = (e.clientX / containerWidth) * 100
    
    // 限制宽度范围在 10% - 50% 之间
    if (newWidth >= 10 && newWidth <= 50) {
        leftWidth.value = newWidth
    }
}

const stopResize = () => {
    isResizing.value = false
    document.removeEventListener('mousemove', handleResize)
    document.removeEventListener('mouseup', stopResize)
    document.body.style.cursor = ''
    document.body.style.userSelect = ''
}

// 获取仪器类型图标
const getInsTypeIcon = (insType) => {
    const icons = {
        'Digitizer': '📊',
        'Fgen': '📡',
        'DCPwr': '⚡',
        'Transceiver': '📻'
    }
    return icons[insType] || '🔧'
}

// 属性按钮点击 - 获取属性值
const onAttrClick = async (attr, channel) => {
    console.log('点击属性:', attr.attr_id, '当前值:', attr.value)
    
    if (!selectedIns.value?.instance) {
        console.error('❌ 仪器实例不存在')
        return
    }
    
    try {
        // 使用通用 get 方法获取属性值，channel 暂时用 0
        const result = await selectedIns.value.instance.getAttributeVi(channel, attr)
        console.log('✅ 获取属性值成功:', { attr_id: attr.attr_id, resultCode: result.resultCode, value: result.value })
        
        // // 更新属性值（响应式）
        // if (result.resultCode === 0) {
        //     attr.value = result.value
        // }
    } catch (error) {
        console.error('❌ 获取属性值失败:', error)
    }
}

// 属性设置 - 设置属性值
const onAttrSet = async (attr, channel) => {
    const newValue = setValues.value[attr.attr_id]
    console.log('设置属性:', attr.attr_id, '新值:', newValue)
    
    if (newValue === undefined || newValue === '') {
        console.warn('⚠️ 请输入要设置的值')
        return
    }
    
    if (!selectedIns.value?.instance) {
        console.error('❌ 仪器实例不存在')
        return
    }
    
    try {
        // 根据类型转换值
        let parsedValue = newValue
        if (attr.ValueType === 'i32' || attr.ValueType === 'u32') {
            parsedValue = parseInt(newValue, 10)
        } else if (attr.ValueType === 'f64') {
            parsedValue = parseFloat(newValue)
        } else if (attr.ValueType === 'bool') {
            parsedValue = newValue === 'true' || newValue === '1'
        }
        
        const resultCode = await selectedIns.value.instance.setAttributeVi(channel, attr, parsedValue)
        console.log('✅ 设置属性值结果:', { attr_id: attr.attr_id, resultCode, value: parsedValue })
        
        // 设置成功后清空输入框并提示
        if (resultCode === 0) {
            // setValues.value[attr.attr_id] = ''
            showToast(`属性 ${attr.attr_id} 设置成功`, 'success')
        } else {
            showToast(`属性 ${attr.attr_id} 设置失败 (错误码: ${resultCode})`, 'error')
        }
    } catch (error) {
        console.error('❌ 设置属性值失败:', error)
        showToast(`设置失败: ${error.message || '未知错误'}`, 'error')
    }
}

// 选择仪器（实例已在初始化时获取）
const selectInstrument = async (ins) => {
    console.log('🔍 选择仪器:', ins.name, '实例:', ins.instance ? '✅' : '❌')
    selectedIns.value = ins
    
    // 如果有实例，获取详细信息
    if (ins.instance) {
        loading.value = true
        try {
            const info = await ins.instance.getInstrumentInfo('*IDN?')
            insInfo.value = info
            console.log('✅ 仪器信息:', info)
            console.log('📊 attributeMap:', ins.instance.iviClassWrapper?.attributeMap)
        } catch (error) {
            console.error('❌ 获取仪器信息失败:', error)
            insInfo.value = null
        } finally {
            loading.value = false
        }
    } else {
        insInfo.value = null
    }
}

onMounted(async () => {
    loading.value = true
    try {
        // 1. 获取仪器列表
        const list = await mainCaller.getInstrumentList()
        console.log('✅ 获取仪器列表:', list.length, '个')
        
        // 2. 并行获取所有仪器实例
        console.log('📡 开始获取所有仪器实例...')
        const listWithInstances = await Promise.all(
            list.map(async (ins) => {
                try {
                    const instance = await mainCaller.getInstrument(String(ins.id))
                    // 等待 init 完成
                    await instance.init()
                    console.log(`  ✅ ${ins.name} 实例已获取, 类型: ${instance.insType}`)
                    return { ...ins, instance }
                } catch (error) {
                    console.warn(`  ⚠️ ${ins.name} 实例获取失败:`, error.message)
                    return { ...ins, instance: null }
                }
            })
        )
        
        instrumentList.value = listWithInstances
        console.log('🎉 所有仪器实例获取完成')
    } catch (error) {
        console.error('❌ 初始化失败:', error)
    } finally {
        loading.value = false
    }
})
</script>

<style scoped lang="less">
.container {
    display: flex;
    width: 100vw;
    height: 100vh;
    position: relative;
    overflow: hidden;

    .left-panel {
        height: 100%;
        background-color: #1a1a1a;
        border-right: 1px solid #333;
        padding: 16px;
        box-sizing: border-box;
        overflow-y: auto;
        flex-shrink: 0;

        h2 {
            color: #fff;
            margin: 0 0 16px 0;
            font-size: 18px;
        }

        .instrument-list {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        
        .loading-state {
            display: flex;
            align-items: center;
            gap: 12px;
            padding: 16px;
            color: #a0a0a0;
            
            .spinner {
                width: 18px;
                height: 18px;
                border: 2px solid #555;
                border-top-color: #6b9fff;
                border-radius: 50%;
                animation: spin 0.8s linear infinite;
            }
        }
    }
    
    .resizer {
        width: 4px;
        height: 100%;
        background: #333;
        cursor: col-resize;
        flex-shrink: 0;
        transition: background 0.2s;
        
        &:hover {
            background: #6b9fff;
        }
        
        &:active {
            background: #4a7dff;
        }
    }

    .right-panel {
        flex: 1;
        height: 100%;
        padding: 24px;
        box-sizing: border-box;
        overflow: auto;
        background-color: #f8f9fa;
        min-width: 0;
        width: 0; // 配合 flex: 1 使用，确保正确计算宽度

        .instrument-detail {
            width: 100%;
            max-width: 100%;
            
            h1 {
                margin: 0 0 24px 0;
                color: #1a1a1a;
                font-size: 24px;
            }

            .detail-section {
                width: 100%;
                background: #fff;
                border-radius: 8px;
                padding: 20px;
                box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
                margin-bottom: 16px;
                
                &:last-child {
                    margin-bottom: 0;
                }

                h3 {
                    margin: 0 0 16px 0;
                    color: #333;
                    font-size: 16px;
                    padding-bottom: 12px;
                    border-bottom: 1px solid #eee;
                }

                .detail-item {
                    display: flex;
                    padding: 10px 0;
                    border-bottom: 1px solid #f0f0f0;

                    &:last-child {
                        border-bottom: none;
                    }

                    .label {
                        width: 80px;
                        color: #666;
                        flex-shrink: 0;
                    }

                    .value {
                        color: #333;

                        &.mono {
                            font-family: monospace;
                            color: #555;
                        }

                        &.available {
                            color: #22c55e;
                            font-weight: 500;
                        }
                    }
                }
            }
            
            .loading-state {
                display: flex;
                align-items: center;
                gap: 12px;
                padding: 16px 20px;
                background: #f0f9ff;
                border-radius: 8px;
                margin-bottom: 16px;
                color: #0369a1;
                
                .spinner {
                    width: 20px;
                    height: 20px;
                    border: 2px solid #0369a1;
                    border-top-color: transparent;
                    border-radius: 50%;
                    animation: spin 0.8s linear infinite;
                }
            }
            
            .instance-status {
                background: #f0fdf4;
                border: 1px solid #86efac;
                
                h3 {
                    color: #166534;
                    border-bottom-color: #86efac;
                }
                
                .hint {
                    color: #15803d;
                    font-size: 14px;
                    margin: 0;
                }
            }
            
            .attr-section {
                width: 100%;
                
                .attr-grid {
                    width: 100%;
                    display: grid;
                    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
                    gap: 12px;
                }
                
                .attr-card {
                    display: flex;
                    flex-direction: column;
                    background: #f8fafc;
                    border: 1px solid #e2e8f0;
                    border-radius: 10px;
                    overflow: hidden;
                    transition: all 0.2s;
                    
                    &:hover {
                        border-color: #6b9fff;
                        box-shadow: 0 4px 12px rgba(107, 159, 255, 0.15);
                    }
                    
                    .attr-header {
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                        padding: 8px 12px;
                        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                        
                        .attr-id {
                            font-size: 11px;
                            color: rgba(255, 255, 255, 0.9);
                            font-weight: 500;
                        }
                        
                        .attr-name {
                            font-size: 8px;
                            color: rgba(255, 255, 255, 0.9);
                            font-weight: 500;
                        }
                        
                        .attr-type {
                            font-size: 10px;
                            color: rgba(255, 255, 255, 0.7);
                            background: rgba(255, 255, 255, 0.2);
                            padding: 2px 6px;
                            border-radius: 4px;
                            text-transform: uppercase;
                        }
                    }
                    
                    .attr-range {
                        padding: 8px 12px;
                        background: #f1f5f9;
                        border-bottom: 1px solid #e2e8f0;
                        display: flex;
                        flex-wrap: wrap;
                        gap: 8px 16px;
                        
                        .range-item {
                            display: flex;
                            align-items: center;
                            gap: 4px;
                            font-size: 11px;
                            
                            &.options {
                                flex-basis: 100%;
                            }
                            
                            .range-label {
                                color: #64748b;
                            }
                            
                            .range-value {
                                color: #334155;
                                font-family: 'SF Mono', 'Consolas', monospace;
                                font-weight: 500;
                            }
                        }
                    }
                    
                    .attr-body {
                        padding: 12px;
                        display: flex;
                        flex-direction: column;
                        gap: 10px;
                        
                        .attr-display, .attr-set {
                            display: flex;
                            align-items: center;
                            gap: 8px;
                            
                            .label {
                                font-size: 11px;
                                color: #64748b;
                                width: 45px;
                                flex-shrink: 0;
                            }
                        }
                        
                        .attr-display {
                            .value {
                                flex: 1;
                                padding: 6px 10px;
                                background: #e2e8f0;
                                border-radius: 6px;
                                font-size: 14px;
                                font-family: 'SF Mono', 'Consolas', monospace;
                                color: #1e293b;
                                font-weight: 600;
                                min-width: 0;
                                overflow: hidden;
                                text-overflow: ellipsis;
                                white-space: nowrap;
                            }
                        }
                        
                        .attr-set {
                            .attr-input {
                                flex: 1;
                                padding: 6px 10px;
                                border: 1px solid #e2e8f0;
                                border-radius: 6px;
                                font-size: 13px;
                                font-family: 'SF Mono', 'Consolas', monospace;
                                color: #1e293b;
                                background: #fff;
                                min-width: 0;
                                transition: all 0.2s;
                                
                                &::placeholder {
                                    color: #94a3b8;
                                    font-family: inherit;
                                }
                                
                                &:focus {
                                    outline: none;
                                    border-color: #6b9fff;
                                    box-shadow: 0 0 0 3px rgba(107, 159, 255, 0.15);
                                }
                            }
                        }
                        
                        .btn-get, .btn-set {
                            padding: 6px 12px;
                            border: none;
                            border-radius: 6px;
                            font-size: 11px;
                            font-weight: 500;
                            cursor: pointer;
                            transition: all 0.15s;
                            flex-shrink: 0;
                        }
                        
                        .btn-get {
                            background: #e0f2fe;
                            color: #0369a1;
                            
                            &:hover {
                                background: #bae6fd;
                            }
                            
                            &:active {
                                transform: scale(0.96);
                            }
                        }
                        
                        .btn-set {
                            background: #dcfce7;
                            color: #166534;
                            
                            &:hover {
                                background: #bbf7d0;
                            }
                            
                            &:active {
                                transform: scale(0.96);
                            }
                        }
                    }
                }
            }
        }

        .empty-state {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100%;
            color: #999;
            font-size: 16px;
        }
    }
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

// Toast 样式
.toast {
    position: fixed;
    top: 24px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 9999;
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 12px 24px;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
    font-size: 14px;
    font-weight: 500;
    
    &.success {
        background: linear-gradient(135deg, #10b981 0%, #059669 100%);
        color: #fff;
    }
    
    &.error {
        background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
        color: #fff;
    }
    
    &.warning {
        background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
        color: #fff;
    }
    
    .toast-icon {
        font-size: 16px;
    }
    
    .toast-message {
        max-width: 300px;
    }
}

// Toast 动画
.toast-enter-active {
    animation: toast-in 0.3s ease-out;
}

.toast-leave-active {
    animation: toast-out 0.3s ease-in;
}

@keyframes toast-in {
    from {
        opacity: 0;
        transform: translateX(-50%) translateY(-20px);
    }
    to {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
    }
}

@keyframes toast-out {
    from {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
    }
    to {
        opacity: 0;
        transform: translateX(-50%) translateY(-20px);
    }
}
</style>