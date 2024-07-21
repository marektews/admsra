<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faBus, faUser, faCircleInfo, faSave, faChevronLeft } from '@fortawesome/free-solid-svg-icons';

const emit = defineEmits(['back'])
const props = defineProps({
    record: { type: Object },
})

const isOnePilot = ref(false)
const usedStaticIdentifier = ref(false)

const info = ref('')
const bus = reactive({
    distance: "",
    parking_mode: "",
    type: "",
    lp: undefined,
    prefix: "",
    static_identifier: null
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

const phone1Length = computed(() => phone_length(pilot1))
const phone1Format = computed(() => phone_format(pilot1))
const phone1Pattern = computed(() => phone_pattern(pilot1))
const phone2Length = computed(() => phone_length(pilot2))
const phone2Format = computed(() => phone_format(pilot2))
const phone2Pattern = computed(() => phone_pattern(pilot2))
const phone3Length = computed(() => phone_length(pilot3))
const phone3Format = computed(() => phone_format(pilot3))
const phone3Pattern = computed(() => phone_pattern(pilot3))

function phone_length(pilot) {
    return pilot.prefix === "+7" ? 10 : 9
}
function phone_format(pilot) {
    if(phone_length(pilot) === 10)
        return "Wpisz 10 cyfr; format: 1234567890"
    else
        return "Wpisz 9 cyfr; format: 123456789"
}
function phone_pattern(pilot) {
    return pilot.prefix === "+7"
        ? "[0-9]{10}"
        : "[0-9]{9}"
}


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
                SubmitButtonDisabled.value = true

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
                if(!usedStaticIdentifier.value) {
                    // korekta
                    data.bus.static_identifier = null
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
                    else
                        SubmitButtonDisabled.value = false
                })
                .catch(err => {
                    console.error("SRA update error:", err)
                    SubmitButtonDisabled.value = false
                })
            }
        }, false)
    })
})

watch(() => props.record, (nv) => {
    console.log("Watcher props.record", nv)
    // Copy data from props
    bus.lp = nv.bus.lp
    bus.prefix = nv.bus.prefix
    bus.static_identifier = nv.bus.static_identifier
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

    if(bus.static_identifier) {
        usedStaticIdentifier.value = true
    }
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

const SubmitButtonDisabled = ref(false)
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
                <FontAwesomeIcon :icon="faBus" /> Bus
            </h4>

            <input type="hidden" id="sraid" :value="props.record.id" />

            <div>
                <label class="form-label">
                    Liczba porządkowa (ważne w przypadku wielu autokarów z jednego zboru)
                </label>
                <div class="bus-lp-class">
                    <div class="form-check">
                        <input 
                            class="form-check-input" 
                            type="radio" 
                            name="bus_lp" 
                            id="bus-lp-none"
                            v-model="bus.lp"
                            :value="null"
                        >
                        <label class="form-check-label" for="bus-lp-none">
                            Brak, tylko jeden autobus jest zgłoszony przez ten zbór
                        </label>
                    </div>
                    <div class="form-check">
                        <input 
                            class="form-check-input" 
                            type="radio" 
                            name="bus_lp" 
                            id="bus-lp-1"
                            v-model="bus.lp"
                            :value="1"
                        >
                        <label class="form-check-label" for="bus-lp-1">
                            1 (pierwszy)
                        </label>
                    </div>
                    <div class="form-check">
                        <input 
                            class="form-check-input" 
                            type="radio" 
                            name="bus_lp" 
                            id="bus-lp-2"
                            v-model="bus.lp"
                            :value="2"
                        >
                        <label class="form-check-label" for="bus-lp-2">
                            2 (drugi)
                        </label>
                    </div>
                    <div class="form-check">
                        <input 
                            class="form-check-input" 
                            type="radio" 
                            name="bus_lp"
                            id="bus-lp-3"
                            v-model="bus.lp"
                            :value="3"
                        >
                        <label class="form-check-label" for="bus-lp-3">
                            3 (trzeci)
                        </label>
                    </div>
                </div>
            </div>

            <div class="mt-3">
                <label class="form-label" for="bus_static_identifier">
                    <div class="form-check">
                        <input class="form-check-input" type="checkbox" id="flexCheckDefault" v-model="usedStaticIdentifier">
                        <label class="form-check-label" for="flexCheckDefault">
                            Nadany identyfikator autokaru (system nie będzie stosował automatyki z literą, turą i numerem sektora)
                        </label>
                    </div>
                </label>
                <input 
                    v-model="bus.static_identifier" 
                    type="text" 
                    maxlength="6" 
                    class="form-control uppercase" 
                    name="bus_static_identifier" 
                    id="bus_static_identifier"
                    :disabled="!usedStaticIdentifier"
                >
            </div>

            <div class="mt-3">
                <label class="form-label" for="bus_prefix" :class="{'text-muted': usedStaticIdentifier}">
                    Litera na identyfikatorze (tura i sektor obliczane są automatycznie na podstawie rozkładu jazdy)
                </label>
                <input 
                    v-model="bus.prefix" 
                    type="text" 
                    maxlength="1" 
                    class="form-control uppercase" 
                    name="bus_prefix" 
                    id="bus_prefix"
                    :disabled="usedStaticIdentifier"
                >
            </div>

            <div class="mt-3">
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
                    <input class="form-check-input" type="radio" name="bus_parking_mode" id="p2" 
                        :checked="bus.parking_mode === 'not_needed'"
                        @input="bus.parking_mode = 'not_needed'"
                    />
                    <label class="form-check-label" for="p2">Pojazd tylko dowozi pasażerów, odjeżdza i przyjeżdza odebrać ich po programie</label>
                </div>
            </div>


            <h4 class="mt-5">
                <FontAwesomeIcon :icon="faUser" />
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
                                style="max-width: 110pt"
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
                                :maxlength="phone1Length"
                                :pattern="phone1Pattern"
                                :placeholder="phone1Format"
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
                                style="max-width: 110pt"
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
                                :maxlength="phone1Length"
                                :pattern="phone1Pattern"
                                :placeholder="phone1Format"
                                required>
                        </div>
                    </div>
                </div>

                <h5 class="mt-4">Sobota</h5>
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input v-model="pilot2.fn" class="form-control" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input v-model="pilot2.ln" class="form-control" required>
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
                                style="max-width: 110pt"
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
                                :maxlength="phone2Length"
                                :pattern="phone2Pattern"
                                :placeholder="phone2Format"
                                required>
                        </div>
                    </div>
                </div>

                <h5 class="mt-4">Niedziela</h5>
                <div class="row g-3">
                    <div class="col-md-6">
                        <label class="form-label">Imię</label>
                        <input v-model="pilot3.fn" class="form-control" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label">Nazwisko</label>
                        <input v-model="pilot3.ln" class="form-control" required>
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
                                style="max-width: 110pt"
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
                                :maxlength="phone3Length"
                                :pattern="phone3Pattern"
                                :placeholder="phone3Format"
                                required>
                        </div>
                    </div>
                </div>
            </template>

            <h4 class="mt-5">
                <FontAwesomeIcon :icon="faCircleInfo" />
                Dodatkowe informacje
            </h4>

            <textarea class="form-control" rows="6"
                v-model="info" 
            />

            <button 
                type="submit" 
                class="mt-5 btn btn-lg btn-outline-primary"
                :disabled="SubmitButtonDisabled"
            >
                <FontAwesomeIcon :icon="faSave" />
                Zapisz
            </button>
            <button 
                class="mt-5 ms-2 btn btn-lg btn-outline-secondary"
                @click="$emit('back')"
            >
                <FontAwesomeIcon :icon="faChevronLeft" />
                Anuluj
            </button>
        </form>
    </div>
</template>

<style scoped>
.bus-lp-class {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    gap: 2pt 16pt;
}
</style>