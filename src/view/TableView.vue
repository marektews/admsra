<script setup>
import { ref, onMounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faPen, faTrash } from '@fortawesome/free-solid-svg-icons';
import xlsxIcon from '@/components/xlsxIcon.vue'
import DeleteModal from '@/components/DeleteModal.vue'

defineEmits(['edit'])
const records = ref([])
const error = ref(false)

onMounted(() => loadData())

function loadData() {
    fetch('/api/sra/table', {
        method: "GET",
        headers: {
            "Accept": "application/json"
        }
    })
    .then(response => {
        console.log('Get SRA response:', response)
        if(response.status === 200)
            return response.json()
        else
            error.value = true
            return []
    })
    .then(d => records.value = d)
    .catch(err => {
        console.log('Get SRA table error:', err)
        error.value = true
    })
}

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

function parkingMode(pm) {
    if(pm == "not_needed")
        return "NIE"
    else if(pm == "needed")
        return "TAK"
    else
        return pm
}

function busDistance(v) {
    switch(v) {
        case "15km": return "15km"
        case "25km": return "25km"
        case "50km": return "50km"
        case "100km": return "100km"
        case "200km": return "200km"
        case "more200km": return "> 200km"
    }
    return v
}

function onExport() {
    window.location = '/api/sra/export/xlsx'
}

const deletingRecord = ref(undefined)
function onStartDeleteItem(item) {
    console.log('Start delete item:', item)
    deletingRecord.value = item
}
function onDelete() {
    console.log('Do delete SRA item:', deletingRecord.value.id)
    fetch(`/api/sra/delete/${deletingRecord.value.id}`)
    .then(response => {
        if(response.status === 200) {
            deletingRecord.value = null
            records.value = []
            loadData()
        }
    })
    .catch(err => console.error('SRA deleting error:', err))
}
</script>

<template>
    <div v-if="error" class="alert alert-danger">
        <div>Coś niedobrego stało się na serwerze.</div>
        <div>Spróbuj ponownie wczytać stronę.</div>
    </div>
    
    <div v-else-if="records.length === 0" class="loading">
        <div class="spinner-border" role="status" />
        <div>Proszę czekać. Trwa pobieranie danych ...</div>
    </div>

    <template v-else>
        <div>
            <button class="btn btn-outline-primary" @click="onExport">
                <div class="mybtn">
                    <xlsxIcon width="24" height="24" />
                    <div>Eksport do formatu Excela</div>
                </div>
            </button>
        </div>

        <table class="table table-sm table-dark table-hover table-bordered">
            <thead>
                <tr>
                    <th rowspan="2">#</th>
                    <th rowspan="2">Data zgłoszenia</th>
                    <th colspan="3">Zbór</th>
                    <th colspan="3">Bus</th>
                    <th colspan="3">Pilot</th>
                    <th rowspan="2">Info</th>
                    <th rowspan="2">Ops</th>
                </tr>
                <tr>
                    <th>Język</th>
                    <th>Numer</th>
                    <th>Nazwa</th>

                    <th>Typ</th>
                    <th>Trasa</th>
                    <th>Parking</th>

                    <th>Piątek</th>
                    <th>Sobota</th>
                    <th>Niedziela</th>
                </tr>
            </thead>
            <tbody class="table-group-divider">
                <tr v-for="(item, index) in records" :key="index">
                    <td>{{ index + 1 }}</td>

                    <td>{{ item.timestamp }}</td>

                    <td>{{ item.zbor.lang }}</td>
                    <td>{{ item.zbor.number }}</td>
                    <td>{{ item.zbor.name }}</td>

                    <td>{{ busType(item.bus.type) }}</td>
                    <td>{{ busDistance(item.bus.distance) }}</td>
                    <td>{{ parkingMode(item.bus.parking_mode) }}</td>

                    <template v-if="!('pilot2' in item)">
                        <td colspan="3">
                            <div>{{ item.pilot1.fn }} {{ item.pilot1.ln }}</div>
                            <div>{{ item.pilot1.phone }}</div>
                            <div>{{ item.pilot1.email }}</div>
                        </td>
                    </template>
                    <template v-else>
                        <td>
                            <div>{{ item.pilot1.fn }} {{ item.pilot1.ln }}</div>
                            <div>{{ item.pilot1.phone }}</div>
                            <div>{{ item.pilot1.email }}</div>
                        </td>
                        <td>
                            <div>{{ item.pilot2.fn }} {{ item.pilot2.ln }}</div>
                            <div>{{ item.pilot2.phone }}</div>
                            <div>{{ item.pilot2.email }}</div>
                        </td>
                        <td>
                            <div>{{ item.pilot3.fn }} {{ item.pilot3.ln }}</div>
                            <div>{{ item.pilot3.phone }}</div>
                            <div>{{ item.pilot3.email }}</div>
                        </td>
                    </template>

                    <td>{{ item.info }}</td>

                    <td>
                        <div class="btns-layout">
                            <button 
                                class="btn btn-outline-primary" 
                                title="Edycja"
                                @click="$emit('edit', item)"
                            >
                                <FontAwesomeIcon :icon="faPen" />
                            </button>
                            <button
                                class="btn btn-outline-danger"
                                title="Usuń"
                                @click="onStartDeleteItem(item)"
                                data-bs-toggle="modal"
                                data-bs-target="#deleteModal"
                            >
                                <FontAwesomeIcon :icon="faTrash" />
                            </button>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>

        <DeleteModal 
            id="deleteModal"
            :record="deletingRecord"
            @ok="onDelete"
        />
    </template>
</template>

<style scoped>
table {
    font-size: 10pt;
    text-align: center;
    vertical-align: middle;
}

div.mybtn {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 6pt;
}

.loading {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 9pt;
}

.btns-layout {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    gap: 6pt;
}
</style>