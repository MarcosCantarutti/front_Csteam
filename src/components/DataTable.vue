<template>
  <div class="container">
    <div v-if="loading" class="loading-overlay">
      <p class="loading-text">Carregando...</p>
      <img src="../assets/holyjoia.webp" alt="Carregando..." class="spinner" />
    </div>
    
    <div v-else class="data-table">
      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <th>Nível</th>
            <th>Raça</th>
            <th>Classe</th>
            <th>Item Level Equipado</th>
            <th>Item Level Médio</th>
            <th>Tier Set</th>
          </tr>
        </thead>
        <tbody>
          <tr 
            v-for="(item, index) in sortedData" 
            :key="index"
            :class="{
              'highlight-green': index < 5,
              'highlight-red': index >= sortedData.length - 5
            }"
          >
            <td>{{ item.name }}</td>
            <td>{{ item.level }}</td>
            <td>{{ item.race }}</td>
            <td>{{ item.class }}</td>
            <td>{{ item.itemLevel.equipped }}</td>
            <td>{{ item.itemLevel.average }}</td>
            <td>
              <ul>
                <li v-for="(tier, index) in item.tierSet" :key="index">
                  {{ tier.slot }}: {{ tier.level }}
                </li>
              </ul>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const data = ref([]);
const loading = ref(true);

const fetchData = async () => {
  try {
    const response = await fetch('https://api-node-csteam.onrender.com/', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json'
      },
      mode: 'cors'
    });

    if (!response.ok) {
      throw new Error(`Erro HTTP: ${response.status}`);
    }

    const fetchedData = await response.json();
    data.value = fetchedData.sort((a, b) => b.itemLevel.average - a.itemLevel.average);
  } catch (error) {
    console.error('Erro ao buscar dados:', error);
  } finally {
    loading.value = false;
  }
};

const sortedData = computed(() => data.value);

onMounted(fetchData);
</script>

<style scoped>
.container {
  margin: 0 auto;
  font-family: Arial, sans-serif;
  position: relative;
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
  background: rgba(0, 0, 0, 0.5); 
  z-index: 1000;
}

.loading-text {
  margin-top: 10px;
  margin-right: 10px;
  font-size: 16px;
  color: #f1f1f1;
}

.spinner {
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #ffffff;
}

.data-table table {
  width: 100%;
  border-collapse: collapse;
}

.data-table th, .data-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

.data-table th {
  background-color: #d3d3d3;
  font-weight: bold;
}

.data-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.data-table tr:hover {
  background-color: #f1f1f1;
}

.data-table tr.highlight-green {
  background-color: #e0f9e0;
}

.data-table tr.highlight-red {
  background-color: #f9e0e0;
}

.data-table ul {
  padding: 0;
  margin: 0;
  list-style: none;
}

.data-table li {
  margin: 2px 0;
}

/* Estilos responsivos */
@media (max-width: 768px) {
  .data-table th, .data-table td {
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
  .data-table th, .data-table td {
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
</style>
