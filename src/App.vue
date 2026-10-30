<template>
    <div class="studio-layout">
        <!-- Left: 3D Product Viewport-->
         <main class="viewport-section">
            <Viewer3D />
         </main>

         <!-- Right: Configuration Panel -->
          <aside class="config-section">
            <header class="studio-header">
                <h1>Product Studio</h1>
                <p> enterprise 3D configuration tool</p>
            </header>
            <div class="config-body">
                <section class="config-card">
                    <h3>Material Customization</h3>
                    <p class="hint">Select finish options below to preview changes in real-time.</p>
                    <!-- configuration controls will go here -->
                </section>
            </div>
          </aside>
    </div>
</template>

<script setup lang="ts">
import Viewer3D from './components/Viewer3D.vue';
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
    grid-template-columns: 1fr 400px;
    width: 100%;
    height: 100vh;

    @media (max-width: 1024px) {
        grid-template-columns: 1fr;
        grid-template-rows: 60vh 1fr;
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
        margin-bottom: 2rem;

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
        gap: 1.5rem;

        .config-card {
            background-color: #0f172a;
            border: 1px solid #334155;
            border-radius: 8px;
            padding: 16px;

            h3 {
                font-size: 1rem;
                font-weight: 600;
                margin-bottom: 8px;
            }

            .hint {
                font-size: 0.8125rem;
                color: #94a3b8;
            }
        }
    }
}
</style>