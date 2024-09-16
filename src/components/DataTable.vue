<template>
  <div class="container">
    <div v-if="loading" class="loading-overlay">
      <img src="../assets/zulpog.webp" alt="Carregando..." class="spinner" />
      <p class="loading-text">Carregando...</p>
      <img src="../assets/holyjoia.webp" alt="Carregando..." class="spinner" />
    </div>

    <div v-else class="data-table">
      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <!-- <th>Nível</th> -->
            <!-- <th>Raça</th> -->
            <th>Classe</th>
            <th>Item Level Equipado / Item Level Médio</th>
            <th>Tier Set</th>
            <th>Itens Encantados</th> 
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
            <!-- <td>{{ item.level }}</td> -->
            <!-- <td>{{ item.race }}</td> -->
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
              <button @click="toggleDetails(index)">
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
  TRINKET_2: 'Trinket 2'
};

const data = ref([]);
const loading = ref(true);
const expandedIndex = ref(null);
const expandedTierIndex = ref(null);

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
    // console.log(fetchedData);
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

const totalEnchantedItems = (items) => {
  let count = 0;
  items.forEach(element => {
    if(element.enchanted){
      count++
    }
  });

  return count;
}

onMounted(fetchData);
</script>

<style scoped>
.container {
  margin: 0 auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  position: relative;
  max-width: 1200px;
  padding: 20px;
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
  color: #105e05; /* verde */
  text-shadow: 
    -1px -1px 2px #000,  
     1px -1px 2px #000,
    -1px  1px 2px #000,
     1px  1px 2px #000;
     font-weight: bold;
     font-size: 18px;
}

</style>
