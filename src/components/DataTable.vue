<template>
  <div :class="{ 'dark-mode': isDarkMode }" class="container">
    <div v-if="loading" class="loading-overlay">
      <img src="../assets/zulpog.webp" alt="Carregando..." class="spinner" />
      <p class="loading-text">{{ textLoading }}</p>
      <img src="../assets/holyjoia.webp" alt="Carregando..." class="spinner" />
    </div>

    <div v-else class="data-table">
      <button @click="() => toggleDarkMode()" class="dark-mode-toggle">
        {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
      </button>
      <button
        @click="() => refreshData()"
        :disabled="isRefreshing"
        class="refresh-button"
      >
        Refresh
      </button>
      <span class="response-text">{{ responseText }}</span>
      <table>
        <thead>
          <tr>
            <th>Name  ({{totalPlayers}})</th>
            <th>Class</th>
            <th>Item lvl Equip/Avg  ({{mediaEquip}}/{{ mediaAvg }})</th>
            <th>Tier Set </th>
            <th>Enchants ({{itensEnchant}}/{{ totalItensEnchant }})</th>
            <th>Embellish. ({{itensEmbelish}}/{{ totalItensEmbelish }})</th>
            <th>Sockets</th>
            <th>Mythic+</th>
            <th>Raid</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in sortedData"
            :key="index"
          >
            <td>
              <b>{{ item.name }}</b>
            </td>
            <td
              :class="{
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
                'class-evoker': item.class === 'Evoker',
              }"
            >
              {{ item.class }}
            </td>
            <td>
              <b
                >{{ item.itemLevel.equipped }} / {{ item.itemLevel.average }} </b
              >
            </td>
            <td>
              <div v-if="item.tierSet.length">
                <b>Tier {{ item.tierSet.length }}/5</b>
              </div>
              <button
              class="btn"
                v-if="item.tierSet.length"
                @click="() => toggleTierDetails(index)"
              >
                {{
                  expandedTierIndex === index
                    ? 'Ocultar'
                    : 'Exibir'
                }}
              </button>
              <div v-if="expandedTierIndex === index">
                <ul>
                  <li v-for="(tier, index) in item.tierSet" :key="index">
                    <div class="enchanted-item">
                      <span
                        >{{ slotNames[tier.slot] || tier.slot }}:
                        {{ tier.level }}</span
                      >
                    </div>
                  </li>
                </ul>
              </div>
            </td>
            <td>
              <div v-if="item.enchantedItems.length">
                <b
                  >Total: {{ totalEnchantedItems(item.enchantedItems) }}/{{
                    item.enchantedItems.length
                  }}</b
                >
              </div>
              <button
              class="btn"
                v-if="item.enchantedItems.length"
                @click="() => toggleDetailsEnchanting(index)"
              >
                {{
                  expandedIndex === index
                    ? 'Ocultar'
                    : 'Exibir'
                }}
              </button>
              <div v-if="expandedIndex === index">
                <ul>
                  <li
                    v-for="(enchantedItem, index) in item.enchantedItems"
                    :key="index"
                  >
                    <div class="enchanted-item">
                      <span
                        >{{
                          slotNames[enchantedItem.slot] || enchantedItem.slot
                        }}:
                      </span>
                      <span
                        :class="
                          enchantedItem.enchanted
                            ? 'enchanted-yes'
                            : 'enchanted-no'
                        "
                      >
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
              <div v-else>
                <b>Total: 0</b>
              </div>
              <button
              class="btn"
                v-if="item.embellishedItems.length"
                @click="() => toggleEmbellishedDetails(index)"
              >
                {{
                  expandedEmbellishedIndex === index
                    ? 'Ocultar'
                    : 'Exibir'
                }}
              </button>
              <div v-if="expandedEmbellishedIndex === index">
                <ul>
                  <li
                    v-for="(
                      embellishedItem, embellishedIndex
                    ) in item.embellishedItems"
                    :key="embellishedIndex"
                  >
                    <div class="enchanted-item">
                      <div v-if="embellishedItem.spells.length">
                        <ul>
                          <li
                            v-for="(
                              spell, spellIndex
                            ) in embellishedItem.spells"
                            :key="spellIndex"
                          >
                            <b
                              ><span
                                >{{
                                  slotNames[embellishedItem.slot] ||
                                  embellishedItem.slot
                                }}
                                - {{ embellishedItem.level }}</span
                              ></b
                            ><br />
                            {{ spell }}
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
                <b
                  >Total: {{ totalSockets(item.sockets) }}/{{
                    item.sockets.length
                  }}</b
                >
              </div>
              <div v-else>
                <b>Total: 0</b>
              </div>
              <button
              class="btn"
                v-if="item.sockets.length"
                @click="() => toggleSocketsDetails(index)"
              >
                {{
                  expandedSocketsIndex === index
                    ? 'Ocultar'
                    : 'Exibir'
                }}
              </button>
              <div v-if="expandedSocketsIndex === index">
                <ul>
                  <li
                    v-for="(socket, socketIndex) in item.sockets"
                    :key="socketIndex"
                  >
                    <div class="socket-item">
                      <span>{{ slotNames[socket.slot] || socket.slot }}: </span>
                      <span>{{ socket.level }}</span>
                      <ul>
                        <li
                          v-for="(socketDetail, detailIndex) in socket.sockets"
                          :key="detailIndex"
                        >
                          <div class="socket-detail">
                            <span
                              >{{ socketDetail.type.name }} -
                              {{ socketDetail.gem }}</span
                            >
                          </div>
                        </li>
                      </ul>
                    </div>
                  </li>
                </ul>
              </div>
            </td>
            <td>
              <div v-if="item.mythicDungeonsDoneThisWeek">
                <b>This week: {{ item.mythicDungeonsDoneThisWeek }}</b>
              </div>
              <div v-else>
                <b>This week: 0</b>
              </div>
              <div v-if="item.mythicDungeonsDoneAlltime">
                <b>All time: {{ item.mythicDungeonsDoneAlltime }}</b>
              </div>
              <div v-if="item.mythicPlusRating.length">
                <b>Score: {{ Math.round(item.mythicPlusRating[0])  }}</b>
              </div>
              <div v-else>
                <b>Score: 0</b>
              </div>
            </td>
            <td>
              <div v-if="item.raidProgress">
                <ul>
                        <li
                          v-for="(item, index) in item.raidProgress"
                          :key="index"
                        >
                            <b>{{ item.summary}}</b>
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
import axios from 'axios';
import { supabase } from '../supabaseCliente.js'; 

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
  NECK: 'Neck',
};

const data = ref([]);
const loading = ref(true);
const expandedIndex = ref(null);
const expandedTierIndex = ref(null);
const expandedEmbellishedIndex = ref(null);
const expandedSocketsIndex = ref(null);
const responseText = ref('');

const isDarkMode = ref(false);
const isRefreshing = ref(false); 

// totais
const totalPlayers = ref(0);
const mediaEquip = ref(0);
const mediaAvg = ref(0);
const itensEnchant = ref(0);
const totalItensEnchant = ref(0);
const itensEmbelish = ref(0);
const totalItensEmbelish = ref(0);

const textLoading = ref('Buscando dados...');

const loadDarkModePreference = () => {
  const darkMode = localStorage.getItem('darkMode');
  if (darkMode) {
    isDarkMode.value = JSON.parse(darkMode);
  }
};

const saveDarkModePreference = () => {
  localStorage.setItem('darkMode', JSON.stringify(isDarkMode.value));
};

const fetchData = async () => {
  loading.value = true;
  try {
    // const response = await axios.get('https://api-node-csteam.onrender.com/dados', {
    //     headers: {
    //       'Content-Type': 'application/json',
    //     },
    //   });

    //   const fetchedData = response.data;

    const { data: playersData, error } = await supabase
      .from('PLAYERS_JSON')
      .select('player_data');

    let JsonPlayersData = playersData.map(item => item.player_data);

    data.value = JsonPlayersData.sort(
      (a, b) => b.itemLevel.average - a.itemLevel.average
    );
    totalPlayers.value = data.value.length;
    
  } catch (error) {
    console.error('Erro ao buscar dados:', error);
  } finally {
    loading.value = false;
  }
};

const refreshData = async () => {
  if (confirm('Deseja atualizar os dados da planilha?')) {
    isRefreshing.value = true; // Desabilita o botão de refresh
    loading.value = true; // Exibe a tela de carregamento
    textLoading.value = 'Atualizando dados...'
    try {

      const response = await axios.get('https://api-node-csteam.onrender.com/', {
        headers: {
          'Content-Type': 'application/json',
        },
      });

      const fetchedData = response.data;
      textLoading.value = 'Buscando itens bi do bi...';
      responseText.value = `${fetchedData.message} - ${formatarHoraAtual()}`;

    } catch (error) {
      console.error('Erro ao buscar dados:', error);
    } finally {
      loading.value = false;
      isRefreshing.value = false;

      fetchData();
    }
  }
};


const sortedData = computed(() => data.value);

const toggleDetailsEnchanting = (index) => {
  expandedIndex.value = expandedIndex.value === index ? null : index;
};

const toggleTierDetails = (index) => {
  expandedTierIndex.value = expandedTierIndex.value === index ? null : index;
};

const toggleEmbellishedDetails = (index) => {
  expandedEmbellishedIndex.value =
    expandedEmbellishedIndex.value === index ? null : index;
};

const totalSockets = (sockets) => {
  return sockets.length;
};

const toggleSocketsDetails = (index) => {
  expandedSocketsIndex.value =
    expandedSocketsIndex.value === index ? null : index;
};

const totalEnchantedItems = (items) => {
  let count = 0;
  items.forEach((element) => {
    if (element.enchanted) {
      count++;
    }
  });

  return count;
};

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
  saveDarkModePreference();
};

const formatarHoraAtual = () => {
  const agora = new Date();

  // Formatar a data e a hora
  const formato = new Intl.DateTimeFormat('pt-BR', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false, // Definido como false para usar o formato de 24 horas
  });

  return formato.format(agora);
};

const calcMediaEquip = () => {
  const totalEquip = data.value.reduce((sum, player) => sum + player.itemLevel.equipped, 0);
  const totalAvg = data.value.reduce((sum, player) => sum + player.itemLevel.average, 0);

  // Calcula as médias
  mediaEquip.value = (totalEquip / data.value.length).toFixed(2);
  mediaAvg.value = (totalAvg / data.value.length).toFixed(2);
};

const calcItensEnchant = () => {
  let totalEncantados = 0;
  let totalItens = 0;

  data.value.forEach(player => {
    totalEncantados += player.enchantedItems.filter(item => item.enchanted).length;
    totalItens += player.enchantedItems.length;
  });

  itensEnchant.value = totalEncantados;
  totalItensEnchant.value = totalItens;
};

const calcItensEmbelish = () => {
  let totalEmbelezados = 0;
  let totalItens = totalPlayers.value * 2;

  data.value.forEach(player => {
    totalEmbelezados += player.embellishedItems.length;
  });

  itensEmbelish.value = totalEmbelezados;
  totalItensEmbelish.value = totalItens;
};

onMounted(async () => {
  loadDarkModePreference();
  await fetchData();
  calcMediaEquip();
  calcItensEnchant();
  calcItensEmbelish();
});
</script>

<style scoped>

.btn{
  padding: 1px;
  cursor: pointer;
}

table{
  border-spacing: 0;
  cursor: pointer;
}
/* Botão de alternância */
.dark-mode-toggle {
  position: absolute;
  top: 5px;
  left: 20px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #4caf50;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
}

.dark-mode-toggle:hover {
  background-color: #45a049;
}

.dark-mode.container {
  /* background-color: #121212; */
  color: #fff;
}

.dark-mode .data-table {
  background-color: #1e1e1e;
}

.dark-mode .data-table th,
.dark-mode .data-table td {
  border: 1px solid #444;
  padding: 2px;
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
  background-color: #444;
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
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
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
  font-size: 15px;
}

.data-table th {
  background-color: #4caf50;
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
  color: #228b22;
  font-weight: bold;
}

.enchanted-no {
  color: #ff2800;
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

.response-text {
  position: absolute;
  top: 4px;
  left: 270px;
  font-size: 22px;
  font-weight: bold;
  color: #0070de;
}

.class-warrior {
  color: #c79c6e; /* Marrom */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-paladin {
  color: #f58cba; /* Rosa */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-hunter {
  color: #abd473; /* Verde */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-rogue {
  color: #fff569; /* Amarelo */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-priest {
  color: #ffffff; /* Branco */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-deathknight {
  color: #c41f3b; /* Vermelho */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-shaman {
  color: #0070de; /* Azul */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-mage {
  color: #69ccf0; /* Azul claro */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-warlock {
  color: #9482c9; /* Roxo */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-monk {
  color: #00ff96; /* Verde neon */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-druid {
  color: #ff7d0a; /* Laranja */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}
.class-demonhunter {
  color: #a330c9; /* Roxo escuro */
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}

.class-evoker {
  color: #015c2a;
  text-shadow: -1px -1px 2px #000, 1px -1px 2px #000, -1px 1px 2px #000,
    1px 1px 2px #000;
  font-weight: bold;
  font-size: 18px;
}

.refresh-button {
  position: absolute;
  top: 5px;
  left: 150px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #ff9800;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
}

.refresh-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}


/* Responsive Styles */
@media (max-width: 835px) {

  .container{
    padding: 20px 0;
  }

  table{
    zoom: 0.75;
  }
  .data-table th,
  .data-table td {
    font-size: 12px;
    padding: 8px;
  }

  .loading-text {
    font-size: 14px;
  }

  .spinner {
    width: 40px;
    height: 40px;
  }

  .response-text {
    font-size: 10px;
  }
}

@media (max-width: 599px) {

  .dark-mode-toggle {
    font-size: 12px;
  }

  .refresh-button {
    font-size: 12px;
  }

  table{
    zoom: 0.75;
  }

  .data-table th,
  .data-table td {
    font-size: 10px;
    padding: 1px;
  }

  .loading-text {
    font-size: 12px;
  }

  .spinner {
    width: 30px;
    height: 30px;
  }
}



</style>
