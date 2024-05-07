<template>
    <div class="bg-red-50 p-2 mb-4">
        Last event: <b>{{ lastEvent }}</b>
        <input type="text" v-model="demo" />
    </div>
    <div class="flex gap-x-4">
        <div class="w-1/6 px-2">
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
                        <i class="fat fa-file" v-if="node.isLeaf"></i>
                        <i class="fat fa-file-invoice" v-if="!node.isLeaf"></i>
                    </span>
                    {{ node.title }}
                </template>
                <template #toggle="{ node }">
                    <span v-if="!node.isLeaf && node.children.length > 0" class="mr-1 cursor-pointer">
                        <i v-if="node.isExpanded" class="far fa-minus"></i>
                        <i v-if="!node.isExpanded" class="far fa-plus"></i>
                    </span>
                    <span v-else></span>
                </template>

                <template #sidebar="{ node }">
                    <span class="ml-2 cursor-pointer" @click="(event) => toggleVisibility(event, node)">
                        <i v-if="!node.data || node.data.visible !== false" class="far fa-eye"></i>
                        <i v-if="node.data && node.data.visible === false" class="far fa-eye-slash"></i>
                    </span>
                </template>
            </XTree>
        </div>

        <div class="w-full">
            <div class="bg-slate-100 text-xs p-8  overflow-y-auto h-[calc(100vh-4rem)]">
                <pre>{{ JSON.stringify(nodes, null, 4) }}</pre>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { XTree } from './components'
import type { Context, TreeNode } from './components'
const demo = ref("asasd")
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
</style>
