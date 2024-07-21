<script setup>
import { ref } from 'vue'
import TableView from './view/TableView.vue'
import RecordEditView from './view/RecordEditView.vue'

const edited = ref(undefined)
</script>

<template>
    <header>
        <div class="title">Ankiety autokarów</div>
    </header>

    <main>
        <RecordEditView v-if="edited !== undefined" 
            :record="edited" 
            @back="edited = undefined"
        />
        <TableView v-else 
            @edit="edited = $event" 
        />
    </main>
</template>

<style>
.uppercase {
    text-transform: uppercase;
}
</style>

<style scoped>
header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16pt;
    margin-bottom: 24pt;
}
.title {
    font-size: 2rem;
}

main {
    min-width: 60vw;
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    gap: 9pt;
}

main > div {
    flex-grow: 1;
}

</style>
