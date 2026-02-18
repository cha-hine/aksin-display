<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import axios from '@/lib/strapi'
import socket from '@/lib/socket-io'
import csvData from '@/assets/time_run_sdn_formatted.csv?raw'

import HelanesTicker from '@/components/HelanesTicker.vue'
import NamazEvent from '@/components/NamazEvent.vue'
import SlideDisplay from '@/components/SlideDisplay.vue'
//import EventsList from '@/components/front/EventsList.vue'
//import NamazTimes from '@/components/front/NamazTimes.vue'

const prayerTimes = ref([])

const formatJamat = (time) => time ? time.substring(0, 5) : ''



const events = ref(['Cours de Coran - Samedi 18h00', 'Conférence islamique - Vendredi'])

const affichage = ref(null)
const helanes = ref([])
const slides = ref([])
//const events = ref([])

const loading = ref(true)
const error = ref(null)

const loadPrayerTimes = () => {
  const now = new Date()
  const day = String(now.getDate()).padStart(2, '0')
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const today = `${day}/${month}`


  const lines = csvData.trim().split('\n')
  for (let i = 1; i < lines.length; i++) {
    const cols = lines[i].split(';')
    if (cols[0] === today) {
      prayerTimes.value = [
        { name: 'Soubh', awwal: cols[1], jamat: formatJamat(affichage.value.data.namaz_jamat_fajr) },
        { name: 'Zohrain', awwal: cols[3], jamat: formatJamat(affichage.value.data.namaz_jamat_zohrain) },
        { name: 'Magribain', awwal: cols[5], jamat: formatJamat(affichage.value.data.namaz_jamat_magribain) },
      ]
      break
    }
  }
}

const fetchAffichage = async () => {
  const res = await axios.get('/affichage')
  affichage.value = res.data || []
  loadPrayerTimes()
}

const fetchHelanes = async () => {
  const res = await axios.get('/helanes')
  helanes.value = res.data || null
}

const fetchSlides = async () => {
  const res = await axios.get('/slides', {
    params: {
      populate: 'image',
    },
  })

  slides.value = res.data.data || []
  console.log('Slides fetched:', slides.value)
}
/*
const fetchEvents = async () => {
  const res = await pb.collection('events').getList({
    filter: 'etat = true',
    sort: 'date',
  })
  events.value = res
}

const fetchNamaz = async () => {
  const res = await pb.collection('namaz').getFullList({
    filter: 'etat = true',
    sort: '-date',
  })
  namaz.value = res[0] || null
}
*/
// ✅ Enregistrer les listeners au niveau du script setup
socket.on('connect', () => {
  console.log('[socket] connected', socket.id)
})

socket.on('connect_error', (err) => {
  console.log('[socket] connect_error', err.message, err)
})

socket.on('update:affichage', () => {
  fetchAffichage()
})

socket.on('update:helane', () => {
  fetchHelanes()
})

socket.on('create:helane', () => {
  fetchHelanes()
})

socket.on('update:slide', () => {
  console.log("test update:slide event received")
  fetchSlides()
})

socket.on('create:slide', () => {
  fetchSlides()
})

onMounted(async () => {
  loading.value = true
  error.value = null

  // Charger les données initiales
  try {
    await Promise.all([fetchAffichage(), fetchHelanes(), fetchSlides()])

  } catch (e) {
    error.value = e?.message ?? String(e)
  } finally {
    loading.value = false
  }
})

// ✅ Nettoyer les listeners
onBeforeUnmount(() => {
  socket.off('connect')
  socket.off('connect_error')
  socket.off('update:affichage')
  socket.off('update:helane')
  socket.off('create:helane')
  socket.off('update:slide')
  socket.off('create:slide')
  socket.offAny()
})
</script>

<template>
  <div
    v-if="affichage"
    class="home-view"
    :style="{
      backgroundColor: '#e9dead',
      width: '1920px',
      height: '1080px',
      padding: '0',
      margin: '0',
    }"
  >
    <p v-if="loading">Chargement…</p>
    <p v-if="error" style="color: red">{{ error }}</p>
    <template v-if="!loading && !error">
      <div
        v-if="affichage.data.namaz_programme"
        class="namaz-wrapper"
        :class="{ 'namaz-wrapper--with-ticker': affichage.data.helane }"
      >
        <NamazEvent
          :prayer-times="prayerTimes"
          :events="events"
          :hijri_date="affichage.data.hijri_date"
          :hijri_date_decalage="affichage.data.hijri_date_decalage"
          :pre_texte_date="affichage.data.pre_texte_date"
        />
      </div>
      <HelanesTicker v-if="affichage.data.helane" :data="helanes.data" />

      <div class="slide-wrapper" :class="{ 'slide-wrapper--centered': !affichage.data.helane }">
        <SlideDisplay :slides="slides" />
      </div>

      <!--       <EventsList v-if="config.events?.enabled !== false" :data="events" :config="config.events" />

      <NamazTimes v-if="config.namaz?.enabled !== false" :data="namaz" :config="config.namaz" /> -->
    </template>
  </div>

  <div v-else-if="!loading">Aucune configuration active.</div>
</template>

<style scoped>
.namaz-wrapper {
  position: absolute;
  right: 0;
  top: 0;
  height: 1080px;
  z-index: 1;
  display: flex;
  align-items: center;
}

.namaz-wrapper--with-ticker {
  height: 1000px;
}

.slide-wrapper--centered {
  height: 1080px;
  display: flex;
  align-items: center;
}
</style>
