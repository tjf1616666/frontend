<template>
    <div class="instrument-card" :class="{ active, 'has-instance': instance }" @click="$emit('click')">
        <div class="instrument-info">
            <div class="instrument-name">
                {{ name }}
                <span v-if="instance" class="instance-badge" title="已获取仪器实例">🔗</span>
            </div>
            <div class="instrument-meta">
                <span class="instrument-model">{{ model }}</span>
                <span class="instrument-type" v-if="insType">{{ insType }}</span>
            </div>
            <div class="instrument-address">{{ address }}</div>
        </div>
        <div class="instrument-status" :class="{ available: status === '可用' }">
            {{ status }}
        </div>
    </div>
</template>

<script setup>
defineProps({
    name: {
        type: String,
        default: 'RIGOL DSU2041'
    },
    model: {
        type: String,
        default: 'DSU2041'
    },
    insType: {
        type: String,
        default: ''
    },
    address: {
        type: String,
        default: 'TCPIP::172.18.26.59::5025::SOCKET'
    },
    status: {
        type: String,
        default: '可用'
    },
    active: {
        type: Boolean,
        default: false
    },
    // 仪器实例（InstrumentCaller）
    instance: {
        type: Object,
        default: null
    }
})

defineEmits(['click'])
</script>

<style scoped lang="less">
.instrument-card {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #2a2a2a;
    border-radius: 8px;
    padding: 16px 20px;
    min-width: 300px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
    cursor: pointer;
    transition: all 0.2s ease;
    border: 2px solid transparent;

    &:hover {
        background-color: #333;
    }

    &.active {
        border-color: #6b9fff;
        background-color: #2d3748;
    }
    
    &.has-instance {
        border-left: 3px solid #22c55e;
    }

    .instrument-info {
        display: flex;
        flex-direction: column;
        gap: 4px;

        .instrument-name {
            color: #ffffff;
            font-size: 16px;
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 6px;
            
            .instance-badge {
                font-size: 14px;
                opacity: 0.9;
            }
        }

        .instrument-meta {
            display: flex;
            align-items: center;
            gap: 8px;

            .instrument-model {
                color: #a0a0a0;
                font-size: 13px;
            }

            .instrument-type {
                color: #6b9fff;
                font-size: 12px;
                padding: 2px 6px;
                background-color: rgba(107, 159, 255, 0.15);
                border-radius: 4px;
            }
        }

        .instrument-address {
            color: #707070;
            font-size: 12px;
            font-family: monospace;
        }
    }

    .instrument-status {
        padding: 4px 12px;
        border-radius: 4px;
        font-size: 13px;
        color: #888;
        border: 1px solid #555;
        background-color: transparent;

        &.available {
            color: #4ade80;
            border-color: #4ade80;
        }
    }
}
</style>

