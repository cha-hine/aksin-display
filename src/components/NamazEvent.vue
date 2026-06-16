<template>
  <div
    class="prayer-display"
    :style="{
      '--text-color': design.textColor,
      '--namaz-card-bg': design.namazCardBg,
      '--namaz-card-border': design.namazCardBorder,
      '--prayer-card-bg': design.prayerCardBg,
      '--prayer-card-border': design.prayerCardBorder,
      '--prayer-header-bg': design.prayerHeaderBg,
      '--prayer-header-border': design.prayerHeaderBorder,
      '--cell-divider': design.cellDivider,
    }"
  >
    <div class="clock-section">
      <div class="time-wrapper">
        <template v-for="(item, idx) in clockItems" :key="idx">
          <span v-if="item.type === 'separator'" class="colon">:</span>
          <div v-else class="digit-slot">
            <Transition name="digit">
              <span :key="item.current" class="digit-face">{{ item.current }}</span>
            </Transition>
          </div>
        </template>
      </div>
      <div class="clock-divider"></div>
      <div class="date-display">{{ formattedDate }}</div>
      <div>
        <span v-if="props.pre_texte_date" class="date-display">{{ props.pre_texte_date }}</span>
        <span v-if="props.hijri_date" class="date-display">{{ hijriDate }}</span>
      </div>
    </div>
    <div class="prayer-times">
      <!-- <h2>NAMAZ TIME</h2> -->
      <div class="prayer-table">
        <div v-for="prayer in prayerTimes" :key="prayer.name" class="prayer-card">
          <div class="prayer-name-header">{{ prayer.name }}</div>
          <div class="prayer-times-row">
            <div class="prayer-time-cell">
              <div class="time-value">
                <p class="time-title">Awwal Time</p>
                {{ prayer.awwal }}
              </div>
            </div>
            <div class="prayer-time-cell">
              <div class="time-value">
                <p class="time-title">Jamat</p>
                {{ prayer.jamat }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- <div class="events-section" v-if="events.length > 0">
      <h3>📅 Programme a venir</h3>
      <div v-for="(event, index) in events" :key="index" class="event-item">
        {{ event }}
      </div>
    </div> -->
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from 'vue'

const props = defineProps({
  hijri_date: {
    type: Boolean,
    default: false,
  },
  hijri_date_decalage: {
    type: Number,
    default: 0,
  },
  pre_texte_date: {
    type: String,
    default: '',
  },
  prayerTimes: {
    type: Array,
    required: true,
    default: () => [],
  },
  events: {
    type: Array,
    default: () => [],
  },
  design: {
    type: Object,
    default: () => ({}),
  },
})

const HIJRI_MONTHS = [
  'Muharram',
  'Safar',
  'Rabi Al-Awwal',
  'Rabi Al-Thani',
  'Jamaadi Al-Awwal',
  'Jamaadi Al-Thani',
  'Rajab',
  "Sha'baan",
  'Ramadhân',
  'Shawwal',
  'Zilkaad',
  'Zilhajj',
]

const gregorianDate = ref('')
const hijriDate = ref('')

const getHijriDate = (offsetDays = 0) => {
  let date = new Date()
  if (offsetDays !== 0) {
    date.setDate(date.getDate() + offsetDays)
  }

  let day = date.getDate()
  let month = date.getMonth()
  let year = date.getFullYear()

  if (year < 1900) year += 1900

  let m = month + 1
  let y = year
  if (m < 3) {
    y -= 1
    m += 12
  }

  let a = Math.floor(y / 100)
  let b = 2 - a + Math.floor(a / 4)
  let jd = Math.floor(365.25 * (y + 4716)) + Math.floor(30.6001 * (m + 1)) + day + b - 1524.5

  // Epoch Hijri : 1er Muharram 1 AH = JD 1948439.5
  const hijriEpoch = 1948439.5

  // Jours depuis l'époque Hijri
  let daysSinceEpoch = Math.floor(jd - hijriEpoch)

  // Années bissextiles dans un cycle de 30 ans : 2, 5, 7, 10, 13, 16, 18, 21, 24, 26, 29
  const leapYears = [2, 5, 7, 10, 13, 16, 18, 21, 24, 26, 29]

  // Fonction pour vérifier si une année Hijri est bissextile
  const isHijriLeapYear = (hYear) => leapYears.includes(hYear % 30)

  // Jours dans une année Hijri (354 ou 355)
  const daysInHijriYear = (hYear) => (isHijriLeapYear(hYear) ? 355 : 354)

  // Jours dans un mois Hijri (mois impairs = 30, pairs = 29, sauf mois 12 en année bissextile = 30)
  const daysInHijriMonth = (hMonth, hYear) => {
    if (hMonth === 12 && isHijriLeapYear(hYear)) return 30
    return hMonth % 2 === 1 ? 30 : 29
  }

  // Calculer l'année Hijri
  let hYear = 1
  let remainingDays = daysSinceEpoch

  while (remainingDays >= daysInHijriYear(hYear)) {
    remainingDays -= daysInHijriYear(hYear)
    hYear++
  }

  // Calculer le mois Hijri
  let hMonth = 1
  while (remainingDays >= daysInHijriMonth(hMonth, hYear)) {
    remainingDays -= daysInHijriMonth(hMonth, hYear)
    hMonth++
  }

  // Le jour (1-indexé)
  let hDay = remainingDays + 1

  return {
    day: hDay,
    monthName: HIJRI_MONTHS[hMonth - 1],
    year: hYear,
    full: `${hDay} ${HIJRI_MONTHS[hMonth - 1]} ${hYear}`,
  }
}

const updateDates = () => {
  // Date Grégorienne
  const now = new Date()
  const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }
  gregorianDate.value = now.toLocaleDateString('fr-FR', options)

  // Date Hijri avec le décalage actuel
  const hijri = getHijriDate(props.hijri_date_decalage)
  hijriDate.value = hijri.full
}

// Watcher pour surveiller les changements de hijri_date_decalage
watch(
  () => props.hijri_date_decalage,
  () => {
    updateDates()
  },
)

const currentTime = ref(new Date())
const hours = ref(0)
const minutes = ref(0)
const seconds = ref(0)
let interval = null

// Animated digit clock items: 6 digits + 2 separators for HH:MM:SS
const clockItems = ref([
  { type: 'digit', current: '0' },
  { type: 'digit', current: '0' },
  { type: 'separator' },
  { type: 'digit', current: '0' },
  { type: 'digit', current: '0' },
  { type: 'separator' },
  { type: 'digit', current: '0' },
  { type: 'digit', current: '0' },
])

// Positions of digits in "HH:MM:SS" string → indices 0,1,3,4,6,7
// These map 1:1 to clockItems indices 0,1,3,4,6,7
const DIGIT_POSITIONS = [0, 1, 3, 4, 6, 7]

const updateClockDigits = () => {
  const timeStr = currentTime.value.toLocaleTimeString('fr-FR', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
  })
  DIGIT_POSITIONS.forEach((pos) => {
    clockItems.value[pos].current = timeStr[pos]
  })
}

const formattedDate = computed(() => {
  return currentTime.value.toLocaleDateString('fr-FR', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
})

const updateTime = () => {
  currentTime.value = new Date()
  hours.value = currentTime.value.getHours() % 12
  minutes.value = currentTime.value.getMinutes()
  seconds.value = currentTime.value.getSeconds()
  updateClockDigits()
}

onMounted(() => {
  updateDates()
  updateTime()
  interval = setInterval(updateTime, 1000)
})

onBeforeUnmount(() => {
  if (interval) {
    clearInterval(interval)
  }
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@500;700&display=swap');

.prayer-display {
  font-family: 'Oswald', sans-serif;
  background: var(--namaz-card-bg, rgba(255, 255, 255, 0.2));
  z-index: 1;
  width: 575px;
  height: 950px;
  border-radius: 35px;
  border: 1px solid var(--namaz-card-border, rgba(0, 0, 0, 0.4));
  display: flex;
  flex-direction: column;
  color: var(--text-color, #000000);
  overflow-y: auto;
}

.clock-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 6px;
  margin-top: 10px;
  gap: 3px;
}

/* Animated digit clock */
.time-wrapper {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 0;
  letter-spacing: 2px;
}

.digit-slot {
  position: relative;
  width: 64px;
  height: 104px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.colon {
  font-family: 'Oswald', sans-serif;
  font-size: 78px;
  line-height: 104px;
  font-weight: 500;
  color: var(--text-color, #000000);
  margin: 0 2px;
  opacity: 0.6;
  padding-bottom: 4px;
}

.digit-face {
  font-family: 'Oswald', sans-serif;
  font-size: 92px;
  line-height: 104px;
  font-weight: 700;
  color: var(--text-color, #000000);
  display: block;
}

/* Subtle fade + slight vertical slide */
.digit-enter-active,
.digit-leave-active {
  transition:
    transform 0.28s ease,
    opacity 0.28s ease;
  position: absolute;
}

.digit-enter-from {
  transform: translateY(28%);
  opacity: 0;
}

.digit-enter-to {
  transform: translateY(0);
  opacity: 1;
}

.digit-leave-from {
  transform: translateY(0);
  opacity: 1;
}

.digit-leave-to {
  transform: translateY(-28%);
  opacity: 0;
}

/* Thin decorative rule between time and date */
.clock-divider {
  width: 72%;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent 0%,
    currentColor 30%,
    currentColor 70%,
    transparent 100%
  );
  opacity: 0.22;
  margin: 2px 0;
}

.date-display {
  font-family: 'Oswald', sans-serif;
  font-size: 38px;
  font-weight: 600;
  letter-spacing: 1px;
  color: var(--text-color, #000000);
  margin: 10px 0;
  text-align: center;
  opacity: 0.82;
}

.prayer-times {
  border-radius: 15px;
  padding: 6px 20px;
  margin-bottom: 4px;
  backdrop-filter: blur(10px);
}

.prayer-times h2 {
  text-align: center;
  font-weight: 900;
  margin: 0 0 30px 0;
  color: var(--text-color, #000000);
  font-size: 55px;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.prayer-table {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.prayer-card {
  background: var(--prayer-card-bg, rgba(255, 255, 255, 0.3));
  border-radius: 12px;
  overflow: hidden;
  border: 2px solid var(--prayer-card-border, rgba(0, 0, 0, 0.2));
  transition: all 0.3s ease;
}

.prayer-card:hover {
  transform: translateX(-5px);
  border-color: var(--namaz-card-border, rgba(0, 0, 0, 0.4));
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2);
}

.prayer-name-header {
  background: var(--prayer-header-bg, rgba(0, 0, 0, 0.15));
  padding: 3px;
  text-align: center;
  font-family: 'Oswald', sans-serif;
  font-size: 38px;
  font-weight: 700;
  color: var(--text-color, #000000);
  text-transform: uppercase;
  letter-spacing: 2px;
  border-bottom: 2px solid var(--prayer-header-border, rgba(0, 0, 0, 0.2));
}

.prayer-times-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0;
}

.prayer-time-cell {
  padding: 6px;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.prayer-time-cell:first-child {
  border-right: 1px solid var(--cell-divider, rgba(0, 0, 0, 0.2));
}

.time-value {
  font-family: 'Oswald', sans-serif;
  font-size: 52px;
  color: var(--text-color, #000000);
  font-weight: 700;
  text-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.time-title {
  font-family: 'Oswald', sans-serif;
  font-size: 24px;
  font-weight: 500;
  letter-spacing: 1px;
  text-transform: uppercase;
  margin: 0;
  padding: 0;
}

.events-section {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  padding: 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(0, 0, 0, 0.1);
  overflow-y: auto;
}

.events-section h3 {
  text-align: center;
  margin: 0 0 20px 0;
  color: #4750ff;
  font-size: 32px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.event-item {
  padding: 10px;
  margin-bottom: 8px;
  background: rgba(92, 71, 255, 0.2);
  border-radius: 8px;
  border-left: 3px solid #5947ff;
  font-size: 24px;
  color: var(--text-color, #000000);
}

.event-item:last-child {
  margin-bottom: 0;
}

.events-section::-webkit-scrollbar {
  width: 6px;
}

.events-section::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}

.events-section::-webkit-scrollbar-thumb {
  background: #ff4757;
  border-radius: 10px;
}
</style>
