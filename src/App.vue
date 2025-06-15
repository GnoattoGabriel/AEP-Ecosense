<template>
  <div id="app">
    <header class="header">
      <img :src="logo" alt="EcoSense" class="logo" />
      <p>Monitoramento urbano de árvores vulneráveis</p>
    </header>
    <main class="main expanded-layout">
      <section class="map-section full-width">
        <h2 class="section-title"></h2>
        <div id="map" class="map"></div>
        <p class="map-tip">Clique no mapa para registrar uma árvore em risco.</p>
      </section>

      <div class="columns">
        <section class="form-section">
          <h2 class="section-title">Registrar Árvore</h2>
          <div class="form-card">
            <input v-model="novaArvore.localizacao" placeholder="📍 Localização" />
            <textarea v-model="novaArvore.condicao" placeholder="📝 Condição observada"></textarea>
            <button @click="registrarArvore">Salvar Registro</button>
          </div>
        </section>

        <section class="list-section">
          <h2 class="section-title">Registros Recentes</h2>
          <ul>
            <li v-for="(arvore, index) in arvores" :key="index">
              <strong>{{ arvore.localizacao }}</strong> — {{ arvore.condicao }}
            </li>
          </ul>
        </section>
      </div>

      <div class="alerta" v-if="chuvaPrevista">
        ⚠️ Atenção: Previsão de chuva – risco de queda de árvores.
      </div>
    </main>
  </div>
</template>

<script setup>
import logo from './assets/ecosense.png';
import { onMounted, ref } from 'vue'
import L from 'leaflet'

const arvores = ref([])
const novaArvore = ref({ localizacao: '', condicao: '' })
const chuvaPrevista = ref(true)

let map

onMounted(() => {
  lucide.createIcons()
  map = L.map('map').setView([-23.4208, -51.9339], 13) // Maringá

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(map)

  map.on('click', e => {
    const { lat, lng } = e.latlng
    novaArvore.value.localizacao = `Lat: ${lat.toFixed(4)}, Lng: ${lng.toFixed(4)}`
    novaArvore.value.condicao = 'Galhos pendendo sobre a via'

    L.marker([lat, lng])
      .addTo(map)
      .bindPopup('🌳 Árvore em risco registrada!')
      .openPopup()
  })
})

function registrarArvore() {
  if (novaArvore.value.localizacao && novaArvore.value.condicao) {
    arvores.value.push({ ...novaArvore.value })
    novaArvore.value.localizacao = ''
    novaArvore.value.condicao = ''
  } else {
    alert('Preencha todos os campos.')
  }
}
</script>

<style scoped>
.logo {
  height: 250px;
  object-fit: contain;
  margin-right: 1rem;
}

.main.expanded-layout {
  max-width: 1400px;
  margin: 2rem auto;
  padding: 0 2rem;
}

.full-width {
  width: 100%;
  margin-bottom: 2rem;
}

.columns {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
}

.form-section,
.list-section {
  flex: 1 1 48%;
  min-width: 320px;
}

@media (max-width: 768px) {
  .columns {
    flex-direction: column;
  }
}

:root {
  --primary: #1d1d1f;
  --secondary: #6e6e73;
  --accent: #0071e3;
  --bg: #f8f5f0;
  --border-radius: 12px;
  --white: #ffffff;
}

body {
  background-color: #f8f5f0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--bg);
  margin: 0;
  padding: 0;
  color: var(--primary);
}

#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.header {
  text-align: center;
  padding: 2rem 1rem;
  background: var(--bg);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.header h1 {
  font-size: 2.5rem;
  margin: 0;
  font-weight: 600;
}

.header p {
  color: var(--secondary);
  margin-top: 0.5rem;
}

.main {
  max-width: 900px;
  margin: 2rem auto;
  padding: 0 1rem;
  flex-grow: 1;
}

.section-title {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  font-weight: 500;
  color: var(--primary);
}

.map-section,
.form-section,
.list-section {
  margin-bottom: 2rem;
}

.map {
  width: 100%;
  height: 350px;
  border-radius: var(--border-radius);
  border: 1px solid #ccc;
}

.map-tip {
  margin-top: 0.5rem;
  color: var(--secondary);
  font-size: 0.9rem;
}

.form-card {
  background: var(--bg);
  border-radius: var(--border-radius);
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-card input,
.form-card textarea {
  border: 1px solid #ccc;
  border-radius: 12px;
  padding: 0.75rem;
  transition: box-shadow 0.2s
}

.form-card input:focus,
.form-card textarea:focus {
  outline: none;
  box-shadow: 0 0 0 2px #0071e360;
}

.form-card button {
  background: #0071e3;
  color: white;
  border: none;
  padding: 0.75rem;
  font-size: 1rem;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: background 0.2s ease;
}

.form-card button:hover {
  background: #005bb5;
}

.alerta {
  background: #fffbe6;
  color: #8a6d3b;
  border-left: 6px solid #ffbf00;
  padding: 1rem;
  margin-bottom: 2rem;
  border-radius: var(--border-radius);
}

.list-section ul {
  list-style: none;
  padding: 0;
}

.list-section li {
  background: var(--white);
  padding: 1rem;
  border-radius: var(--border-radius);
  margin-bottom: 1rem;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.04);
}
</style>
