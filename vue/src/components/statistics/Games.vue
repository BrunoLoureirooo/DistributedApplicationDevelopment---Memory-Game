<template>
  <div class="flex flex-col items-center justify-center space-y-6">
    <!-- Total Games -->
    <div class="grid-cols-1 grid text-center py-2">
      <h1>Total Games: {{ totalGames }}</h1>
    </div>

    <!-- Filtros para duração e tipo -->
    <div class="flex space-x-4 items-center">
      <div>
        <label for="type" class="mr-2">Select Type:</label>
        <select id="type" v-model="type" @change="fetchGamesPerDay" class="mb-4 mr-2 px-4 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring focus:ring-blue-200 focus:border-blue-300">
          <option value="day">Day</option>
          <option value="week">Week</option>
          <option value="month">Month</option>
        </select>
      </div>

      <div>
        <label for="duration" class="mr-2">Select Duration:</label>
        <select id="duration" v-model="duration" @change="fetchGamesPerDay" class="mb-4 mr-2 px-4 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring focus:ring-blue-200 focus:border-blue-300">
          <option v-for="n in maxDuration" :key="n" :value="n">
            {{ type==='month'?n+1:n }}
          </option>
        </select>
      </div>
    </div>

    <!-- Gráfico de linhas para jogos por dia -->
    <div v-if="gamesPerDay.length > 0" class="w-full">
      <h2 class="font-bold text-center">Games Over Time</h2>
      <Line :data="gamesPerDayChartData" :options="lineChartOptions" />
    </div>
    <p v-else class="text-center">No data available for selected filters.</p>

    <!-- Tabela de jogos por dia -->
    <div v-if="gamesPerDay.length > 0" class="w-full mt-4">
      <h2 class="font-bold text-center">Games Over Time (Table)</h2>
      <table class="table-auto w-full text-left border">
        <thead>
        <tr>
          <th class="px-4 py-2 border">Period</th>
          <th class="px-4 py-2 border">Games Played</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="entry in gamesPerDay" :key="entry.period">
          <td class="px-4 py-2 border">{{ type==='month'? months[entry.period-1] : entry.period }}</td>
          <td class="px-4 py-2 border">{{ entry.total }}</td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- Gráficos de pizza -->
    <div class="flex space-x-4 w-full">
      <!-- Games Per Board -->
      <div v-if="gamesPerBoard.length > 0" class="w-1/2 text-center">
        <h2 class="font-bold">Games Per Board</h2>
        <Pie :data="gamesPerBoardChartData" :options="chartOptions" />
      </div>
      <p v-else>No data available for games per Board.</p>

      <!-- Games Per Gamemode -->
      <div v-if="gamesPerGamemode.length > 0" class="w-1/2 text-center">
        <h2 class="font-bold">Games Per Gamemode</h2>
        <Pie :data="gamesPerGamemodeChartData" :options="chartOptions" />
      </div>
      <p v-else>No data available for games per gamemode.</p>
    </div>

    <!-- Tabelas de jogos por board e gamemode lado a lado -->
    <div class="flex space-x-4 w-full mt-4">
      <!-- Tabela de jogos por board -->
      <div v-if="gamesPerBoard.length > 0" class="w-1/2">
        <h2 class="font-bold text-center">Games Per Board (Table)</h2>
        <table class="table-auto w-full text-left border">
          <thead>
          <tr>
            <th class="px-4 py-2 border">Board</th>
            <th class="px-4 py-2 border">Games Played</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="board in gamesPerBoard" :key="board.board_id">
            <td class="px-4 py-2 border">
              {{ boards.find(b => b.id === board.board_id) ? `Board ${boards.find(b => b.id === board.board_id).board_cols}x${boards.find(b => b.id === board.board_id).board_rows}` : `Board ${board.board_id}` }}
            </td>
            <td class="px-4 py-2 border">{{ board.total }}</td>
          </tr>
          </tbody>
        </table>
      </div>

      <!-- Tabela de jogos por gamemode -->
      <div v-if="gamesPerGamemode.length > 0" class="w-1/2">
        <h2 class="font-bold text-center">Games Per Gamemode (Table)</h2>
        <table class="table-auto w-full text-left border">
          <thead>
          <tr>
            <th class="px-4 py-2 border">Gamemode</th>
            <th class="px-4 py-2 border">Games Played</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="gamemode in gamesPerGamemode" :key="gamemode.type">
            <td class="px-4 py-2 border">{{ gamemode.type === 'S' ? 'Singleplayer' : 'Multiplayer' }}</td>
            <td class="px-4 py-2 border">{{ gamemode.total }}</td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from "axios"
import { Pie, Line } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement, LineElement, PointElement, LinearScale, CategoryScale } from 'chart.js'

// Registrar componentes necessários do Chart.js
ChartJS.register(Title, Tooltip, Legend, ArcElement, LineElement, PointElement, LinearScale, CategoryScale)

const totalGames = ref(0)
const gamesPerBoard = ref([])
const gamesPerGamemode = ref([])
const gamesPerDay = ref([])
const boards = ref([])
const duration = ref(7)
const lastType = ref('day')
const type = ref('day')
const maxDuration = computed(() => {
  if (type.value === 'day') return 30
  if (type.value === 'week') return 12
  if (type.value === 'month') return 11
  return 7
})
const months = ref([
  'January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'
]);

// Funções para buscar dados
const fetchGamesPerDay = async () => {
  if (lastType.value !== type.value){
    duration.value = 3
    lastType.value = type.value
  }
  const response = await axios.get("/games/total/duration", {
    params: {
      duration: duration.value,
      type: type.value
    }
  })
  gamesPerDay.value = response.data
}

const fetchBoards = async () => {
  const response = await axios.get("/boards")
  boards.value = response.data.data
  console.log("Boards fetched:", boards.value)
}

const fetchGamesPerBoard = async () => {
  const response = await axios.get("/games/total/board")
  gamesPerBoard.value = response.data
}

const fetchGamesPerGamemode = async () => {
  const response = await axios.get("/games/total/gamemode")
  gamesPerGamemode.value = response.data
}

const fetchTotalGames = async () => {
  const response = await axios.get("/games/total")
  totalGames.value = response.data.num_games
}

onMounted(() => {
  fetchBoards()
  fetchTotalGames()
  fetchGamesPerBoard()
  fetchGamesPerGamemode()
  fetchGamesPerDay()
})

// Dados para gráfico de linhas
// Dados para gráfico de linhas
const gamesPerDayChartData = computed(() => {
  let labels = gamesPerDay.value.map(entry => entry.period);

  // Se o tipo for 'month', convertemos os números para os nomes dos meses
  if (type.value === 'month') {
    labels = labels.map(monthIndex => months.value[monthIndex - 1]); // Ajusta porque os meses começam do 1 no gráfico
  }

  return {
    labels: labels,
    datasets: [{
      label: 'Games Played',
      data: gamesPerDay.value.map(entry => entry.total),
      borderColor: '#36A2EB',
      backgroundColor: 'rgba(54, 162, 235, 0.2)',
      tension: 0.3,
    }]
  }
})

// Configurações para gráfico de linhas
const lineChartOptions = {
  responsive: true,
  plugins: {
    title: {
      display: true,
      text: 'Games Over Time'
    },
    tooltip: {
      callbacks: {
        label: tooltipItem => `${tooltipItem.raw} games`
      }
    }
  }
}

// Dados para gráficos de pizza
const gamesPerBoardChartData = computed(() => ({
  labels: gamesPerBoard.value.map(board => {
    const boardDetails = boards.value.find(b => b.id === board.board_id)
    return boardDetails ? `Board ${boardDetails.board_cols}x${boardDetails.board_rows}` : `Board ${board.board_id}`
  }),
  datasets: [{
    data: gamesPerBoard.value.map(board => board.total),
    backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#FF5733', '#33FF57', '#5733FF'],
    hoverOffset: 4
  }]
}))

const gamesPerGamemodeChartData = computed(() => ({
  labels: gamesPerGamemode.value.map(gamemode => gamemode.type === 'S' ? 'Singleplayer' : 'Multiplayer'),
  datasets: [{
    data: gamesPerGamemode.value.map(gamemode => gamemode.total),
    backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#FF5733'],
    hoverOffset: 4
  }]
}))

// Configurações para gráficos de pizza
const chartOptions = {
  responsive: true,
  plugins: {
    title: {
      display: true,
      text: 'Games Distribution'
    },
    tooltip: {
      callbacks: {
        label: tooltipItem => `${tooltipItem.label}: ${tooltipItem.raw} games`
      }
    }
  }
}
</script>

<style scoped>
.chart-container {
  width: 48%;
  margin-bottom: 2rem;
}

.flex {
  display: flex;
}

.space-x-4 {
  gap: 16px;
}

.table-auto {
  border-collapse: collapse;
}

th, td {
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

.border {
  border: 1px solid #ddd;
}
</style>
