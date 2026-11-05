<template>
    <div class="studio-layout">
        <!-- Left: 3D Product Viewport-->
         <main class="viewport-section">
            <Viewer3D 
            :active-part="config.activePart"
            :color="currentPartConfig.color"
            :roughness="currentPartConfig.roughness"
            :metalness="currentPartConfig.metalness"
            />
         </main>

         <!-- Right: Configuration Panel -->
          <aside class="config-section">
            <header class="studio-header">
                <h1>Product Studio</h1>
                <p> enterprise 3D configuration tool</p>
            </header>
            <div class="config-body">
                <!-- Part Selection -->
                 <section class="config-card">
                    <h3>Select Component</h3>
                    <div class="part-tabs">
                        <button
                        v-for="part in partsList"
                        :key="part.id"
                        class="tab-btn"
                        :class="{active: config.activePart === part.id}"
                        @click="config.activePart = part.id"
                        >
                    {{ part.name }}
                    </button>
                    </div>
                 </section>

                <!-- Color Selection -->
                <section class="config-card">
                    <h3>Color Palette ({{ currentPartName }})</h3>
                    <div class="color-options">
                        <button
                        v-for="colorOption in colorSwatches"
                        :key="colorOption.hex"
                        class="swatch-btn"
                        :class="{active: currentPartConfig.color === colorOption.hex}"
                        :style="{backgroundColor: colorOption.hex}"
                        :title="colorOption.name"
                        @click="currentPartConfig.color = colorOption.hex"
                        ></button>
                    </div>
                </section>

                <!-- Material Properties -->
                 <section class="config-card">
                    <h3>Surface Finish ({{ currentPartName }})</h3>

                    <div class="control-group">
                        <label>
                            <span>Roughness</span>
                            <span>{{ Math.round(currentPartConfig.roughness * 100) }}%</span>
                        </label>
                        <input
                        type="range"
                        min="0"
                        max="1"
                        step="0.05"
                        v-model.number="currentPartConfig.roughness"
                        />
                    </div>

                    <div class="control-group">
                        <label>
                            <span>Metalness</span>
                            <span>{{ Math.round(currentPartConfig.metalness * 100) }}%</span>
                        </label>
                        <input
                        type="range"
                        min="0"
                        max="1"
                        step="0.05"
                        v-model.number="currentPartConfig.metalness"
                        />
                    </div>
                 </section>
            </div>
          </aside>
    </div>
</template>

<script setup lang="ts">
import { reactive, computed } from 'vue';
import Viewer3D from './components/Viewer3D.vue';

const partsList = [
    { id: 'body', name: 'Main Body'},
    { id: 'speakers', name: 'Speakers & Accents'},
] as const;

const config = reactive({
    activePart: 'body' as 'body' | 'speakers',
    parts: {
        body: {
            color: '#6366f1',
            roughness: 0.3,
            metalness: 0.8,
        },
        speakers: {
            color: '#f59e0b',
            roughness: 0.5,
            metalness: 0.2,
        }
        }
    });

const currentPartConfig = computed(() => config.parts[config.activePart]);
const currentPartName = computed(() => partsList.find(part => part.id === config.activePart)?.name || '');

const colorSwatches = [
    { name: 'Indigo', hex: '#6366f1' },
    { name: 'Emerald', hex: '#10b981' },
    { name: 'Rose', hex: '#f43f5e' },
    { name: 'Amber', hex: '#f59e0b' },
    { name: 'Slate', hex: '#64748b' },
];
</script>

<style lang="scss">
*{
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body, html {
    width: 100%;
    height: 100%;
    font-family: Inter, system-ui, -apple-system, sans-serif;
    background-color: #0f127a;
    color: #f8fafc;
}

#app {
    width: 100%;
    height: 100vh;
}

.studio-layout {
    display: grid;
    grid-template-columns: 1fr 380px;
    width: 100%;
    height: 100vh;

    @media (max-width: 1024px) {
        grid-template-columns: 1fr;
        grid-template-rows: 55vh 1fr;
    }
}

.viewport-section {
    width: 100%;
    height: 100%;
}

.config-section {
    background-color: #1e293b;
    border-left: 1px solid #334155;
    display: flex;
    flex-direction: column;
    padding: 24px;
    overflow-y: auto;

    .studio-header {
        margin-bottom: 1.5rem;

        h1 {
            font-size: 1.5rem;
            font-weight: 700;
            color: #ffffff;
        }

        p {
            font-size: 0.875rem;
            color: #94a3b8;
            margin-top: 4px;
        }
    }

    .config-body {
        display: flex;
        flex-direction: column;
        gap: 1.25rem;

        .config-card {
            background-color: #0f172a;
            border: 1px solid #334155;
            border-radius: 8px;
            padding: 16px;

            h3 {
                font-size: 0.95rem;
                font-weight: 600;
                margin-bottom: 12px;
                color: #e2e8f0;
            }
            
            .part-tabs {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 8px;

                .tab-btn {
                    background-color: #1e293b;
                    border: 1px solid #334155;
                    color: #94a3b8;
                    padding: 10px;
                    border-radius: 6px;
                    font-size: 0.85rem;
                    font-weight: 500;
                    cursor: pointer;
                    transition: all 0.2s ease;

                    &.hover {
                        color: #ffffff;
                        border-color: #475569;
                    }

                    &.active {
                        background-color: #6366f1;
                        border-color: #6366f1;
                        color: #ffffff;
                    }
                }
            }
            .color-options {
                display: flex;
                gap: 12px;

                .swatch-btn {
                    width: 36px;
                    height: 36px;
                    border-radius: 50%;
                    border: 2px solid transparent;
                    cursor: pointer;
                    transition: transform 0.15s ease, border-color 0.15s ease;

                    &.hover {
                        transform: scale(1.1);
                    }

                    &.active {
                        border-color: #ffffff;
                        box-shadow: 0 0 0 2px #6366f1;
                    }
                }
            }

            .control-group {
                display: flex;
                flex-direction: column;
                gap: 8px;

                &:not(:last-child) {
                    margin-bottom: 14px;
                }

                label {
                    display: flex;
                    justify-content: space-between;
                    font-size: 0.85rem;
                    color: #94a3b8;
                }

                input[type="range"]{
                    width: 100%;
                    accent-color: #6366f1;
                    cursor: pointer;
                }
            }
        }
    }
}
</style>