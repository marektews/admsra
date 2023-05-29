<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'

const emit = defineEmits(['back'])
const props = defineProps({
    record: { type: Object },
})

const isOnePilot = ref(false)
const info = ref('')
const bus = reactive({
    distance: "",
    parking_mode: "",
    type: ""
})
const pilot1 = reactive({
    fn: "",
    ln: "",
    email: "",
    prefix: "",
    number: ""
})

const pilot2 = reactive({
    fn: "",
    ln: "",
    email: "",
    prefix: "",
    number: ""
})

const pilot3 = reactive({
    fn: "",
    ln: "",
    email: "",
    prefix: "",
    number: ""
})


onMounted(() => {
    // Fetch all the forms we want to apply custom Bootstrap validation styles to
    const forms = document.querySelectorAll('.needs-validation')

    // Loop over them and prevent submission
    Array.from(forms).forEach(form => {
        form.addEventListener('submit', event => {
            if (!form.checkValidity()) {
                form.reportValidity()
                event.preventDefault()
                event.stopPropagation()
                form.classList.add('was-validated')
            }
            else {
                form.classList.add('was-validated')

                let data = {
                    id: props.record.id,
                    bus: bus,
                    info: info.value,
                    pilot1: pilot1,
                }
                if(!isOnePilot.value) {
                    data.pilot2 = pilot2
                    data.pilot3 = pilot3
                }

                fetch('/api/sra/write', {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify(data),
                })
                .then(response => {
                    if(response.status === 200)
                        emit('back')
                })
                .catch(err => console.error("SRA update error:", err))
            }
        }, false)
    })
})

watch(() => props.record, (nv, _) => {
    console.log("Watcher props.record", nv)
    // Copy data from props
    bus.distance = nv.bus.distance
    bus.parking_mode = nv.bus.parking_mode
    bus.type = nv.bus.type
    initPilotData(pilot1, nv.pilot1)
    if('pilot2' in nv)
        initPilotData(pilot2, nv.pilot2)
    else
        isOnePilot.value = true
    if('pilot3' in nv)
        initPilotData(pilot3, nv.pilot3)
    if(nv.info !== null)
        info.value = nv.info
},
{
    deep: true,
    immediate: true
})

function initPilotData(p, src) {
    let parts = src.phone.split(' ')
    p.fn = src.fn
    p.ln = src.ln
    p.email = src.email
    p.prefix = parts[0]
    p.number = parts[1]
}

</script>

<template>
    <div class="container">
        <h4>
            ({{ props.record.zbor.lang }} - {{ props.record.zbor.number }}) {{ props.record.zbor.name }}
        </h4>

        <form 
            action=""
            method="dialog"
            class="mt-5 needs-validation" 
            novalidate
        >
            <h4>
                <i class="fa-solid fa-bus" />
                Bus
            </h4>

            <input type="hidden" id="sraid" :value="props.record.id" />

            <div>
                <label class="form-label">Typ pojazdu</label>
                <select class="form-select" v-model="bus.type" name="bus_type">
                    <option value="minibus_9">Minibus do 9 osób</option>
                    <option value="minibus_30">Minibus do 30 osób</option>
                    <option value="autokar_50">Autokar do 50 osób</option>
                    <option value="autokar_70">Autokar 60-70 osób</option>
                    <option value="autobus_12m">Autobus miejski - 12m (pojedyńczy)</option>
                    <option value="autobus_18m">Autobus miejski - 18m (przegubowy)</option>
                </select>
            </div>
            <div class="mt-3">
                <label class="form-label">Długość trasy</label>
                <select class="form-select" v-model="bus.distance" name="bus_distance">
                    <option value="15km">do 15 km</option>
                    <option value="25km">do 25 km</option>
                    <option value="50km">do 50 km</option>
                    <option value="100km">do 100 km</option>
                    <option value="200km">do 200 km</option>
                    <option value="more200km">powyżej 200 km</option>
                </select>
            </div>

            <div class="mt-3">
                <label class="form-label">Zapotrzebowanie parkingowe</label>
                <div class="form-check">
                    <input class="form-check-input" type="radio" name="bus_parking_mode" id="p1" 
                        :checked="bus.parking_mode === 'needed'"
                        @input="bus.parking_mode = 'needed'"
                    />
                    <label class="form-check-label" for="p1">Potrzebne miejsce parkingowe</label>
                </div>
                <div class="form-check">
                    <input class="form-check-input" type="radio" name="bus_parking_mode" id="p1" 
                        :checked="bus.parking_mode === 'not_needed'"
                        @input="bus.parking_mode = 'not_needed'"
                    />
                    <label class="form-check-label" for="p2">Pojazd tylko dowozi pasażerów, odjeżdza i przyjeżdza odebrać ich po programie</label>
                </div>
            </div>


            <h4 class="mt-5">
                <i class="fa-solid fa-user" />
                Pilot
            </h4>

            <div class="form-check">
                <input v-model="isOnePilot"
                    class="form-check-input" 
                    type="checkbox" 
                    id="onePilot" 
                    name="one_pilot"
                >
                <label class="form-check-label" for="onePilot">
                    Jeden pilot we wszystkie dni kongresowe
                </label>
            </div>

            <template v-if="isOnePilot">
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input v-model="pilot1.fn" class="form-control" name="p1_fn" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input v-model="pilot1.ln" class="form-control" name="p1_ln" required>
                    </div>
                </div>
                <div class="mt-1 row g-3">
                    <div class="col-md-6">
                        <label class="form-label">E-mail</label>
                        <input 
                            v-model="pilot1.email"
                            class="form-control" 
                            type="email" 
                            required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nr telefonu</label>
                        <div class="input-group">
                            <select 
                                v-model="pilot1.prefix"
                                style="max-width: 90pt"
                                class="form-select"
                                required
                            >
                                <option value="+48">PL: +48</option>
                                <option value="+7">RU: +7</option>
                                <option value="+380">UA: +380</option>
                            </select>
                            <input 
                                v-model="pilot1.number"
                                class="form-control" 
                                type="tel" 
                                required>
                        </div>
                    </div>
                </div>
            </template>

            <template v-else>
                <h5 class="mt-4">Piątek</h5>
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input v-model="pilot1.fn" class="form-control" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input v-model="pilot1.ln" class="form-control" required>
                    </div>
                </div>
                <div class="mt-1 row g-3">
                    <div class="col-md-6">
                        <label class="form-label">E-mail</label>
                        <input 
                            v-model="pilot1.email" 
                            class="form-control" 
                            type="email" 
                            required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nr telefonu</label>
                        <div class="input-group">
                            <select 
                                v-model="pilot1.prefix"
                                style="max-width: 90pt"
                                class="form-select" 
                                required
                            >
                                <option value="+48">PL: +48</option>
                                <option value="+7">RU: +7</option>
                                <option value="+380">UA: +380</option>
                            </select>
                            <input 
                                v-model="pilot1.number"
                                class="form-control" 
                                type="tel" 
                                required>
                        </div>
                    </div>
                </div>

                <h5 class="mt-4">Sobota</h5>
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input class="form-control" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input class="form-control" required>
                    </div>
                </div>
                <div class="mt-1 row g-3">
                    <div class="col-md-6">
                        <label class="form-label">E-mail</label>
                        <input 
                            v-model="pilot2.email"
                            class="form-control" 
                            type="email" 
                            required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nr telefonu</label>
                        <div class="input-group">
                            <select 
                                v-model="pilot2.prefix"
                                style="max-width: 90pt"
                                class="form-select" 
                                required
                            >
                                <option value="+48">PL: +48</option>
                                <option value="+7">RU: +7</option>
                                <option value="+380">UA: +380</option>
                            </select>
                            <input 
                                v-model="pilot2.number"
                                class="form-control" 
                                type="tel" 
                                required>
                        </div>
                    </div>
                </div>

                <h5 class="mt-4">Niedziela</h5>
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input class="form-control" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input class="form-control" required>
                    </div>
                </div>
                <div class="mt-1 row g-3">
                    <div class="col-md-6">
                        <label class="form-label">E-mail</label>
                        <input 
                            v-model="pilot3.email"
                            class="form-control" 
                            type="email" 
                            required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nr telefonu</label>
                        <div class="input-group">
                            <select 
                                v-model="pilot3.prefix"
                                style="max-width: 90pt"
                                class="form-select" 
                                required
                            >
                                <option value="+48">PL: +48</option>
                                <option value="+7">RU: +7</option>
                                <option value="+380">UA: +380</option>
                            </select>
                            <input 
                                v-model="pilot3.number"
                                class="form-control" 
                                type="tel" 
                                required>
                        </div>
                    </div>
                </div>
            </template>

            <h4 class="mt-5">
                <i class="fa-solid fa-circle-info" />
                Dodatkowe informacje
            </h4>

            <textarea class="form-control" rows="6"
                v-model="info" 
            />

            <button 
                type="submit" 
                class="mt-5 btn btn-lg btn-outline-primary"
            >
                <i class="fa-solid fa-save" />
                Zapisz
            </button>
            <button 
                class="mt-5 ms-2 btn btn-lg btn-outline-secondary"
                @click="$emit('back')"
            >
                <i class="fa-solid fa-chevron-left" />
                Anuluj
            </button>
        </form>
    </div>
</template>