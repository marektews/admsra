<script setup>
import { ref, computed } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faTrash } from '@fortawesome/free-solid-svg-icons';

const emit = defineEmits(['ok'])
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

const congregationName = computed(() => {
    const rc = props.record
    if(!rc) return ""
    if(rc.hasbus)
        return rc.hasbus.congregation.name
    else
    if(rc.nobus)
        return rc.nobus.congregation.name
    else
        return ""
})

const bustype = computed(() => {
    const rc = props.record
    if(rc && rc.hasbus)
        return rc.hasbus.bus.type
    else
        return ""
})

const pilotName = computed(() => {
    const rc = props.record
    if(rc && rc.hasbus)
        return `${rc.hasbus.pilot1.fn} ${rc.hasbus.pilot1.ln}`
    else
        return ""
})

const pin = computed(() => {
    const rc = props.record
    if(rc) {
        if(rc.hasbus)
            return rc.hasbus.id.slice(-4)
        else
        if(rc.nobus)
            return rc.nobus.id.slice(-4)
    }
    return ""
})

const isOkEnabled = computed(() => {
    return confirmationData.value === pin.value
})

function onDelete() {
    confirmationData.value = ''
    emit('ok')
}
function onExit() {
    confirmationData.value = ''
}
</script>

<template>
    <div class="modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5>Kasowanie</h5>
                </div>

                <div class="modal-body">
                    <div>
                        <div>Dane do skasowania:</div>
                        <div class="ms-3">
                            <div>Zbor: {{ congregationName }}</div>
                            <div>Bus: {{ busType(bustype) }}</div>
                            <div>Pilot: {{ pilotName }}</div>
                        </div>
                    </div>
                    <h5 class="mt-3">
                        Czy na pewno skasować ten wpis?
                    </h5>
                    <div>
                        <div>Potwierdź wpisując PIN: <b>{{ pin }}</b></div>
                        <div>
                            <input class="form-control" v-model="confirmationData" type="text" />
                        </div>
                    </div>
                </div>

                <div class="modal-footer">
                    <button 
                        type="button"
                        class="btn btn-secondary"
                        data-bs-dismiss="modal"
                        aria-label="Zamknij"
                        @click="onExit"
                    >
                        Zamknij
                    </button>
                    <button
                        type="button"
                        class="btn btn-danger"
                        data-bs-dismiss="modal"
                        :disabled="!isOkEnabled"
                        @click="onDelete"
                    >
                        <FontAwesomeIcon :icon="faTrash" /> Usuń
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>
