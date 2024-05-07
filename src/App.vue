<template>
    <div class="bg-red-500">Last event: {{ lastEvent }}</div>
    <div class="row">
        <div class="col-6">
            <XTree
                v-model="nodes"
                ref="xtreeRef"
                :allow-multiselect="true"
                :max-scroll-speed="10"
                :scroll-area-height="20"
                @select="nodeSelected"
                @drop="nodeDropped"
                @toggle="nodeToggled"
            >
                <template #title="{ node }">
                    <span class="ml-1 text-slate-900">
                        <i class="far fa-file" v-if="node.isLeaf"></i>
                        <i class="far fa-file-invoice" v-if="!node.isLeaf"></i>
                    </span>

                    {{ node.title }}
                </template>
                <template #toggle="{ node }">
                    <span v-if="!node.isLeaf">
                        <i v-if="node.isExpanded" class="far fa-minus"></i>
                        <i v-if="!node.isExpanded" class="far fa-plus"></i>
                    </span>
                </template>

                <template #sidebar="{ node }">
                    <span class="visible-icon" @click="(event) => toggleVisibility(event, node)">
                        <i v-if="!node.data || node.data.visible !== false" class="far fa-eye"></i>
                        <i v-if="node.data && node.data.visible === false" class="far fa-eye-slash"></i>
                    </span>
                </template>
            </XTree>
        </div>

        <div class="col-6">
            <div class="json-preview">
                <pre>{{ JSON.stringify(nodes, null, 4) }}</pre>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { XTree } from './components'
import type { Context, TreeNode } from './components'

interface DataType {
    visible?: boolean
}

const nodes = ref<TreeNode<DataType>[]>([
    { title: 'Item1', isLeaf: true },
    { title: 'Item2', isLeaf: true, data: { visible: false } },
    { title: 'Folder1' },
    {
        title: 'Folder2',
        isExpanded: true,
        children: [
            { title: 'Item3', isLeaf: true },
            { title: 'Item4', isLeaf: true },
            {
                title: 'Folder3',
                children: [{ title: 'Item5', isLeaf: true }],
            },
        ],
    },
    { title: 'Folder5', isExpanded: false },
    { title: 'Item6', isLeaf: true },
    { title: 'Item7', isLeaf: true, data: { visible: false } },
    {
        title: 'Folder6',
        children: [
            {
                title: 'Folder7',
                children: [
                    { title: 'Item8', isLeaf: true },
                    { title: 'Item9', isLeaf: true },
                ],
            },
        ],
    },
])

// data
const selectedNodesTitle = ref('')
const lastEvent = ref('No last event')
const xtreeRef = ref<Context<DataType> | null>(null)

//methods
const nodeSelected = (nodes, event) => {
    selectedNodesTitle.value = nodes.map((node) => node.title).join(', ')
    lastEvent.value = `Select nodes: ${selectedNodesTitle}`
}

const nodeToggled = (node, event) => {
    lastEvent.value = `Node ${node.title} is ${node.isExpanded ? 'expanded' : 'collapsed'}`
}

const nodeDropped = (nodes, position, event) => {
    lastEvent.value = `Nodes: ${nodes
        .map((node) => node.title)
        .join(', ')} are dropped ${position.placement} ${position.node.title}`
}

const removeNode = () => {
    const paths = xtreeRef.value?.getSelected().map((node) => node.path)
    xtreeRef.value?.remove(paths)
}

const toggleVisibility = (event, node) => {
    event.stopPropagation()
    const visible = !node.data || node.data.visible !== false
    xtreeRef.value.updateNode({ path: node.path, patch: { data: { visible: !visible } } })
    lastEvent.value = `Node ${node.title} is ${!visible ? 'visible' : 'invisible'} now`
}

onMounted(() => {
    window.addEventListener('keydown', onArrowDownHandler)
})

onUnmounted(() => {
    window.removeEventListener('keydown', onArrowDownHandler)
})

const onArrowDownHandler = (event) => {
    event.preventDefault()
    const keyCode = event.code

    if (xtreeRef.value.selectionSize === 1) {
        const selectedNode = xtreeRef.value.getSelected()[0]
        let nodeToSelect

        if (keyCode === 'ArrowDown') {
            nodeToSelect = xtreeRef.value.getNextNode(selectedNode.path, (node) => node.isVisible)
        } else if (keyCode === 'ArrowUp') {
            nodeToSelect = xtreeRef.value.getPrevNode(selectedNode.path, (node) => node.isVisible)
        } else if (keyCode === 'Enter' || keyCode === 'Space') {
            if (selectedNode.isLeaf) return
            xtreeRef.value.updateNode({ path: selectedNode.path, patch: { isExpanded: !selectedNode.isExpanded } })
        }

        if (!nodeToSelect) return

        xtreeRef.value.select(nodeToSelect.path)
    } else if (keyCode === 'ArrowDown') {
        xtreeRef.value.select(xtreeRef.value.getFirstNode().path)
    } else if (keyCode === 'ArrowUp') {
        xtreeRef.value.select(xtreeRef.value.getLastNode().path)
    }
}
</script>

<style>
@import '/xtree-min.css';
.row {
    @apply flex;
}

.col-6 {
    flex-grow: 1;
    max-width: 50%;
    position: relative;
    padding-right: 15px;
    padding-left: 15px;
}

.json-preview {
    padding: 1em;
    background-color: #f8f9fa;
    border-radius: 0.25em;
    overflow: auto;
}

.last-event {
    color: white;
    background-color: rgba(100, 100, 255, 0.5);
    padding: 10px;
    border-radius: 2px;
    margin-bottom: 10px;
}
</style>
