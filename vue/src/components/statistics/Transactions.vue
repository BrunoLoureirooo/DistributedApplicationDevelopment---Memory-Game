<template>
  <div class="flex flex-col items-center justify-center">
    <!-- Total Transactions -->
    <div class="grid-cols-1 grid text-center py-2">
      <h1 class="text-xl font-semibold">Total Transactions: {{ transactionsTotal }}</h1>
    </div>

    <!-- Container for the pie charts -->
    <div class="flex space-x-4 w-full">
      <!-- Pie Chart for Transactions Per Payment Method -->
      <div v-if="transactionsPerPaymentMethod.length > 0" class="w-1/2">
        <h2 class="font-bold text-center">Transactions Per Payment Method</h2>
        <Pie :data="transactionsPerPaymentMethodChartData" :options="chartOptions" />
      </div>
      <div v-else class="w-1/2 text-center">
        <p>No data available for transactions per Payment Method.</p>
      </div>

      <!-- Pie Chart for Transactions Per Type -->
      <div v-if="transactionsPerType.length > 0" class="w-1/2">
        <h2 class="font-bold text-center">Transactions Per Type</h2>
        <Pie :data="transactionsPerTypeChartData" :options="chartOptions" />
      </div>
      <div v-else class="w-1/2 text-center">
        <p>No data available for transactions per Type.</p>
      </div>
    </div>

    <!-- Container for the tables -->
    <div class="flex space-x-4 w-full mt-6">
      <!-- Table for Transactions Per Payment Method -->
      <div v-if="transactionsPerPaymentMethod.length > 0" class="w-1/2 overflow-x-auto">
        <h2 class="font-bold text-center">Transactions Per Payment Method - Table</h2>
        <table class="min-w-full table-auto mt-4 bg-white border border-gray-300 rounded-lg shadow-md">
          <thead>
          <tr class="bg-gray-200">
            <th class="py-2 px-4 text-left text-sm font-semibold text-gray-700">Payment Method</th>
            <th class="py-2 px-4 text-left text-sm font-semibold text-gray-700">Total Transactions</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item, index) in transactionsPerPaymentMethod" :key="index">
            <td class="py-2 px-4 text-sm text-gray-800">{{ item.payment_type }}</td>
            <td class="py-2 px-4 text-sm text-gray-800">{{ item.total_transactions }}</td>
          </tr>
          </tbody>
        </table>
      </div>

      <!-- Table for Transactions Per Type -->
      <div v-if="transactionsPerType.length > 0" class="w-1/2 overflow-x-auto">
        <h2 class="font-bold text-center">Transactions Per Type - Table</h2>
        <table class="min-w-full table-auto mt-4 bg-white border border-gray-300 rounded-lg shadow-md">
          <thead>
          <tr class="bg-gray-200">
            <th class="py-2 px-4 text-left text-sm font-semibold text-gray-700">Transaction Type</th>
            <th class="py-2 px-4 text-left text-sm font-semibold text-gray-700">Total Transactions</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item, index) in transactionsPerType" :key="index">
            <td class="py-2 px-4 text-sm text-gray-800">
              {{ item.type === 'P' ? 'Purchase' : item.type === 'B' ? 'Bonus' : 'Internal' }}
            </td>
            <td class="py-2 px-4 text-sm text-gray-800">{{ item.total_transactions }}</td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { Pie } from 'vue-chartjs';
import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement, CategoryScale, LinearScale } from 'chart.js';

// Register necessary chart components
ChartJS.register(Title, Tooltip, Legend, ArcElement, CategoryScale, LinearScale);

const transactionsTotal = ref(0);
const transactionsPerPaymentMethod = ref([]);
const transactionsPerType = ref([]);

const fetchTransactionsTotal = async () => {
  const response = await axios.get('/transactions/total');
  transactionsTotal.value = response.data[0].total_transactions;
};

const fetchTransactionsPerPaymentMethod = async () => {
  const response = await axios.get('/transactions/total/payment_method');
  transactionsPerPaymentMethod.value = response.data;
};

const fetchTransactionsPerType = async () => {
  const response = await axios.get('/transactions/total/type');
  transactionsPerType.value = response.data;
};

onMounted(() => {
  fetchTransactionsTotal();
  fetchTransactionsPerPaymentMethod();
  fetchTransactionsPerType();
});

// Pie chart options
const chartOptions = {
  responsive: true,
  plugins: {
    title: {
      display: true,
      text: 'Transactions Distribution'
    },
    tooltip: {
      callbacks: {
        label: (tooltipItem) => {
          return `${tooltipItem.label}: ${tooltipItem.raw} transactions`;
        }
      }
    }
  }
};

// Pie chart data for Transactions Per Payment Method
const transactionsPerPaymentMethodChartData = computed(() => {
  return {
    labels: transactionsPerPaymentMethod.value.map(paymentMethod => paymentMethod.payment_type),
    datasets: [{
      data: transactionsPerPaymentMethod.value.map(paymentMethod => paymentMethod.total_transactions),
      backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#FF5733'], // Colors for each segment
      hoverOffset: 4
    }]
  };
});

// Pie chart data for Transactions Per Type
const transactionsPerTypeChartData = computed(() => {
  return {
    labels: transactionsPerType.value.map(transactionType => transactionType.type ==='P' ? 'Purchase' : transactionType.type==='B' ? 'Bonus' : 'Internal'),
    datasets: [{
      data: transactionsPerType.value.map(transactionType => transactionType.total_transactions),
      backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#FF5733'], // Colors for each segment
      hoverOffset: 4
    }]
  };
});
</script>

<style scoped>
/* Optional: Add custom styling for your pie charts */
.w-12 {
  width: 48%;
  margin-bottom: 2rem;
}

.flex {
  display: flex;
}

.space-x-4 {
  gap: 16px;
}
</style>
