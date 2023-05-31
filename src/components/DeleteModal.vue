<script setup>
import { ref, computed } from 'vue'

defineEmits(['ok'])
const props = defineProps(['record'])
const confirmationData = ref('')

function busType(bt) {
    switch(bt) {
        case "minibus_9": return "minibus 9"
        case "minibus_30": return "minibus 30"
        case "autokar_50": return "autokar 50"
        case "autokar_70": return "autokar 60-70"
        case "autobus_12m": return "autobus 12m"
        case "autobus_18m": return "autobus 18m"
    }
    return bt
}

const isOkEnabled = computed(() => {
    return confirmationData.value === props.record?.pilot1.email
})
</script>

<template>
    <div class="modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5>Kasowanie</h5>
                </div>

                <div class="modal-body">
                    <p>
                        <div>Dane do skasowania:</div>
                        <div class="ms-3">
                            <div>Zbor: {{ props.record?.zbor.name }}</div>
                            <div>Bus: {{ busType(props.record?.bus.type) }}</div>
                            <div>Pilot: {{ props.record?.pilot1.fn }} {{ props.record?.pilot1.ln }}</div>
                        </div>
                    </p>
                    <p>
                    <b>Czy na pewno skasować ten wpis?</b>
                    </p>
                    <p>
                        <div>Potwierdź wpisując adres email pilota: <b>{{ props.record?.pilot1.email }}</b></div>
                        <div>
                            <input class="form-control" v-model="confirmationData" type="text" />
                        </div>
                    </p>
                </div>

                <div class="modal-footer">
                    <button 
                        type="button"
                        class="btn btn-secondary"
                        data-bs-dismiss="modal"
                        aria-label="Zamknij"
                    >
                        Zamknij
                    </button>
                    <button
                        type="button"
                        class="btn btn-danger"
                        data-bs-dismiss="modal"
                        :disabled="!isOkEnabled"
                        @click="$emit('ok')"
                    >
                        <i class="fa-solid fa-trash" />
                        Usuń
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
@media (prefers-color-scheme: dark) {
    div.modal-content {
        background-color: var(--color-background) !important;
    }

    .btn-close {
        color: var(--color-text) !important;
    }
}
</style>