<template>
    <div>
        <button @click="editItem">編輯</button>
        <button @click="deleteItem">刪除</button>
    </div>
</template>

<script setup lang="ts">
    import { defineProps, defineEmits } from 'vue';

    const props = defineProps({
        item: Object
    });

    const emits = defineEmits(['edit', 'delete']);

    const editItem = () => {
        emits('edit', props.item);
    };

    const deleteItem = () => {
        emits('delete', props.item);
    };
</script>