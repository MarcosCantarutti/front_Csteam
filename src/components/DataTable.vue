<template>
  <div :class="{ 'dark-mode': isDarkMode }" class="container">

    
    <div v-if="loading" class="loading-overlay">
      <img src="../assets/zulpog.webp" alt="Carregando..." class="spinner" />
      <p class="loading-text">Carregando...</p>
      <img src="../assets/holyjoia.webp" alt="Carregando..." class="spinner" />
    </div>

    <div v-else class="data-table">
      <button @click="toggleDarkMode" class="dark-mode-toggle">
      {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
    </button>
      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <th>Classe</th>
            <th>Item Level Equipado / Item Level Médio</th>
            <th>Tier Set</th>
            <th>Itens Encantados</th> 
            <th>Itens Embelezados</th>
            <th>Sockets</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in sortedData"
            :key="index"
            :class="{
              'highlight-green': index < 5,
              'highlight-red': index >= sortedData.length - 5,
            }"
          >
            <td><b>{{ item.name }}</b></td>
            <td :class="{
              'class-warrior': item.class === 'Warrior',
              'class-paladin': item.class === 'Paladin',
              'class-hunter': item.class === 'Hunter',
              'class-rogue': item.class === 'Rogue',
              'class-priest': item.class === 'Priest',
              'class-deathknight': item.class === 'Death Knight',
              'class-shaman': item.class === 'Shaman',
              'class-mage': item.class === 'Mage',
              'class-warlock': item.class === 'Warlock',
              'class-monk': item.class === 'Monk',
              'class-druid': item.class === 'Druid',
              'class-demonhunter': item.class === 'Demon Hunter',
              'class-evoker': item.class === 'Evoker'
            }"
            >{{ item.class }}</td>
            <td><b>{{ item.itemLevel.equipped }} / {{ item.itemLevel.average }}</b></td>
            <td>
              <div v-if="item.tierSet.length">
                  <b>Tier {{ item.tierSet.length }}/5</b>
              </div>
              <button v-if="item.tierSet.length" @click="toggleTierDetails(index)">
                {{ expandedTierIndex === index ? 'Esconder Tier Set' : 'Exibir Tier Set' }}
              </button>
              <div v-if="expandedTierIndex === index">
                
                <ul>
                  <li v-for="(tier, index) in item.tierSet" :key="index">
                    <div class="enchanted-item">
                      <span>{{ slotNames[tier.slot] || tier.slot }}: {{ tier.level }}</span>
                    </div>
                  </li>
                </ul>
              </div>
            </td>
            <td>
              <div v-if="item.enchantedItems.length">
                  <b>Total: {{ totalEnchantedItems(item.enchantedItems) }}/{{item.enchantedItems.length}}</b>
                </div>
              <button v-if="item.enchantedItems.length" @click="toggleDetails(index)">
                {{ expandedIndex === index ? 'Esconder Itens Encantados' : 'Exibir Itens Encantados' }}
              </button>
              <div v-if="expandedIndex === index">
              
                <ul>
                  <li v-for="(enchantedItem, index) in item.enchantedItems" :key="index">
                    <div class="enchanted-item">
                      <span>{{ slotNames[enchantedItem.slot] || enchantedItem.slot }}: </span>
                      <span :class="enchantedItem.enchanted ? 'enchanted-yes' : 'enchanted-no'">
                        {{ enchantedItem.enchanted ? 'Sim' : 'Não' }}
                      </span>
                    </div>
                  </li>
                </ul>
              </div>
            </td>

            <td>
              <div v-if="item.embellishedItems.length">
                <b>Total: {{ item.embellishedItems.length }}</b>
              </div>
              <button v-if="item.embellishedItems.length" @click="toggleEmbellishedDetails(index)">
                {{ expandedEmbellishedIndex === index ? 'Esconder Itens Embelezados' : 'Exibir Itens Embelezados' }}
              </button>
              <div v-if="expandedEmbellishedIndex === index">
                <ul>
                  <li v-for="(embellishedItem, embellishedIndex) in item.embellishedItems" :key="embellishedIndex">
                    <div class="enchanted-item">
                      <div v-if="embellishedItem.spells.length">
                        <ul>
                          <li v-for="(spell, spellIndex) in embellishedItem.spells" :key="spellIndex">
                            <b><span>{{ slotNames[embellishedItem.slot] || embellishedItem.slot }} - {{ embellishedItem.level }}</span></b><br> {{ spell }}
                          </li>
                        </ul>
                      </div>
                    </div>
                  </li>
                </ul>
              </div>
            </td>
            <td>
              <div v-if="item.sockets.length">
                <b>Total: {{ totalSockets(item.sockets) }}/{{ item.sockets.length }}</b>
              </div>
              <button v-if="item.sockets.length" @click="toggleSocketsDetails(index)">
                {{ expandedSocketsIndex === index ? 'Esconder Sockets' : 'Exibir Sockets' }}
              </button>
              <div v-if="expandedSocketsIndex === index">
                <ul>
                  <li v-for="(socket, socketIndex) in item.sockets" :key="socketIndex">
                    <div class="socket-item">
                      <span>{{ slotNames[socket.slot] || socket.slot }}: </span>
                      <span>{{ socket.level }}</span>
                      <ul>
                        <li v-for="(socketDetail, detailIndex) in socket.sockets" :key="detailIndex">
                          <div class="socket-detail">
                            <span>{{ socketDetail.type.name }} - {{ socketDetail.gem }}</span>
                          </div>
                        </li>
                      </ul>
                    </div>
                  </li>
                </ul>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const slotNames = {
  CHEST: 'Chest',
  SHOULDER: 'Shoulder',
  HANDS: 'Hands',
  LEGS: 'Legs',
  FEET: 'Feet',
  WRIST: 'Wrist',
  FINGER_1: 'Ring 1',
  FINGER_2: 'Ring 2',
  BACK: 'Back',
  MAIN_HAND: 'Main Hand',
  OFF_HAND: 'Off Hand',
  HEAD: 'Head',
  SHOULDERS: 'Shoulders',
  TRINKET_1: 'Trinket 1',
  TRINKET_2: 'Trinket 2',
  WAIST: 'Waist',
  NECK: 'Neck'
};

const data = ref([]);
const loading = ref(true);
const expandedIndex = ref(null);
const expandedTierIndex = ref(null);
const expandedEmbellishedIndex = ref(null);
const expandedSocketsIndex = ref(null);

const isDarkMode = ref(false); 

const fetchData = async () => {
  try {
    const response = await fetch('https://api-node-csteam.onrender.com/', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
      },
      mode: 'cors',
    });

    if (!response.ok) {
      throw new Error(`Erro HTTP: ${response.status}`);
    }

    const fetchedData = await response.json();
    console.log(fetchedData);
    data.value = fetchedData.sort(
      (a, b) => b.itemLevel.average - a.itemLevel.average
    );
  } catch (error) {
    console.error('Erro ao buscar dados:', error);
  } finally {
    loading.value = false;
  }
};

const sortedData = computed(() => data.value);

const toggleDetails = (index) => {
  expandedIndex.value = expandedIndex.value === index ? null : index;
};

const toggleTierDetails = (index) => {
  expandedTierIndex.value = expandedTierIndex.value === index ? null : index;
};

const toggleEmbellishedDetails = (index) => {
  expandedEmbellishedIndex.value = expandedEmbellishedIndex.value === index ? null : index;
};

const totalSockets = (sockets) => {
  return sockets.length;
};

const toggleSocketsDetails = (index) => {
  expandedSocketsIndex.value = expandedSocketsIndex.value === index ? null : index;
};

const totalEnchantedItems = (items) => {
  let count = 0;
  items.forEach(element => {
    if(element.enchanted){
      count++
    }
  });

  return count;
}

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
};

onMounted(fetchData);
</script>

<style scoped>
/* Botão de alternância */
.dark-mode-toggle {
  position: absolute;
  top: 5px;
  left:  20px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #4CAF50;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
}

.dark-mode-toggle:hover {
  background-color: #45a049;
}

.dark-mode.container {
  background-color: #121212;
  color: #fff;
}

.dark-mode .data-table {
  background-color: #1e1e1e;
}

.dark-mode .data-table th,
.dark-mode .data-table td {
  border: 1px solid #444;
}

.dark-mode .data-table th {
  background-color: #333;
  color: #e0e0e0;
}

.dark-mode .data-table tr:nth-child(even) {
  background-color: #2c2c2c;
}

.dark-mode .data-table tr:hover {
  background-color: #444;
}

.dark-mode .loading-overlay {
  background: rgba(0, 0, 0, 0.9);
}

.dark-mode .loading-text {
  color: #e0e0e0;
}

.dark-mode .data-table tr.highlight-green {
  background-color: rgba(0, 0, 0, 0.9);
}

.dark-mode .data-table tr.highlight-red {
  background-color:  #444;
}

/* Manter o restante dos estilos como estão */
.data-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #f9f9f9;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin-top: 30px;
}

.container {
  margin: 0 auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  position: relative;
  max-width: 1200px;
  padding: 20px;
  transition: background-color 0.3s, color 0.3s;
  color: #000;
}

.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.7);
  z-index: 1000;
}

.loading-text {
  margin-top: 10px;
  font-size: 18px;
  font-weight: bold;
  color: #f9f9f9;
}

.spinner {
  width: 60px;
  height: 60px;
  margin: 0 10px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #f9f9f9;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.data-table table {
  width: 100%;
}

.data-table th,
.data-table td {
  border: 1px solid #ddd;
  text-align: left;
  transition: background-color 0.3s;
}

.data-table th {
  background-color: #4CAF50;
  color: #fff;
  font-weight: bold;
  padding: 2px;
}

.data-table tr:nth-child(even) {
  background-color: #f2f2f2;
}

.data-table tr:hover {
  background-color: #e0e0e0;
}

.data-table tr.highlight-green {
  background-color: #e0f9e0;
}

.data-table tr.highlight-red {
  background-color: #f9e0e0;
}

.enchanted-yes {
  color: #228B22;
  font-weight: bold;
}

.enchanted-no {
  color: #FF2800;
  font-weight: bold;
}

.data-table ul {
  padding: 0;
  margin: 0;
  list-style: none;
}

.data-table li {
  margin: 2px 0;
}

.enchanted-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.socket-detail {
  font-style: italic;
}

/* Responsive Styles */
@media (max-width: 768px) {
  .data-table th,
  .data-table td {
    font-size: 14px;
    padding: 8px;
  }

  .loading-text {
    font-size: 14px;
  }

  .spinner {
    width: 40px;
    height: 40px;
  }
}

@media (max-width: 480px) {
  .data-table th,
  .data-table td {
    font-size: 12px;
    padding: 6px;
  }

  .loading-text {
    font-size: 12px;
  }

  .spinner {
    width: 30px;
    height: 30px;
  }
}

.class-warrior {
  color: #C79C6E; /* Marrom */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-paladin {
  color: #F58CBA; /* Rosa */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-hunter {
  color: #ABD473; /* Verde */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-rogue {
  color: #FFF569; /* Amarelo */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-priest {
  color: #FFFFFF; /* Branco */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-deathknight {
  color: #C41F3B; /* Vermelho */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-shaman {
  color: #0070DE; /* Azul */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-mage {
  color: #69CCF0; /* Azul claro */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-warlock {
  color: #9482C9; /* Roxo */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-monk {
  color: #00FF96; /* Verde neon */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-druid {
  color: #FF7D0A; /* Laranja */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}
.class-demonhunter {
  color: #A330C9; /* Roxo escuro */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}

.class-evoker {
  color: #015c2a;
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}


</style>
