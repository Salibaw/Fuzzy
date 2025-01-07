<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-blue-600 via-indigo-500 to-purple-600 text-gray-800">
    <div class="bg-white shadow-2xl rounded-2xl p-10 max-w-lg w-full transform transition-transform hover:scale-105">
      <h1 class="text-5xl font-extrabold text-center mb-8 text-indigo-600">Sistem Fuzzy Produksi</h1>
      <div class="flex flex-col gap-6">
        <div>
          <label for="demand" class="block text-lg font-medium text-gray-700 mb-2">Jumlah Permintaan:</label>
          <input
            type="number"
            v-model="demand"
            id="demand"
            class="border border-indigo-300 rounded-lg p-3 w-full focus:ring focus:ring-indigo-400 focus:border-indigo-500 shadow-sm"
          />
        </div>
        <div>
          <label for="stock" class="block text-lg font-medium text-gray-700 mb-2">Jumlah Persediaan:</label>
          <input
            type="number"
            v-model="stock"
            id="stock"
            class="border border-indigo-300 rounded-lg p-3 w-full focus:ring focus:ring-indigo-400 focus:border-indigo-500 shadow-sm"
          />
        </div>
        <button
          @click="calculateProduction"
          class="bg-indigo-600 text-white font-bold py-3 rounded-lg w-full hover:bg-indigo-700 transition-transform transform hover:scale-105 shadow-md"
        >
          Hitung Produksi
        </button>
        <button
          @click="resetForm"
          class="bg-gray-300 text-gray-700 font-bold py-3 rounded-lg w-full mt-4 hover:bg-gray-400 transition-transform transform hover:scale-105 shadow-md"
        >
          Reset
        </button>
      </div>
      <div v-if="production !== null" class="mt-8 text-center">
        <h2 class="text-2xl font-semibold text-gray-800">
          Produksi yang Disarankan:
          <span class="text-indigo-600 text-3xl font-bold">{{ production }}</span>
        </h2>
      </div>
      <div v-if="production !== null" class="mt-10">
        <canvas id="chart" width="400" height="200"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
Chart.register(...registerables);

export default {
  data() {
    return {
      stock: 0,
      demand: 0,
      production: null,
      chart: null,
    };
  },
  methods: {
    calculateProduction() {
      const stockLevel = this.fuzzyStock(this.stock);
      const demandLevel = this.fuzzyDemand(this.demand);
      this.production = this.defuzzify(stockLevel, demandLevel);
      this.updateChart();
    },
    fuzzyStock(stock) {
      if (stock <= 300) return 'Minim';
      else if (stock <= 400) return 'Sedang';
      else return 'Banyak';
    },
    fuzzyDemand(demand) {
      if (demand <= 3000) return 'Rendah';
      else if (demand <= 4000) return 'Sedang';
      else return 'Tinggi';
    },
    defuzzify(stockLevel, demandLevel) {
      const productionRules = {
        Minim: { Rendah: 'Kecil', Sedang: 'Tidak Produksi', Tinggi: 'Tidak Produksi' },
        Sedang: { Rendah: 'Kecil', Sedang: 'Kecil', Tinggi: 'Tidak Produksi' },
        Banyak: { Rendah: 'Sedang', Sedang: 'Kecil', Tinggi: 'Besar' },
      };

      return productionRules[stockLevel][demandLevel];
    },
    initializeChart() {
      const ctx = document.getElementById('chart').getContext('2d');
      this.chart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: ['Stok', 'Permintaan', 'Produksi'],
          datasets: [
            {
              label: 'Nilai',
              data: [this.stock, this.demand, 0],
              backgroundColor: ['#4f46e5', '#9333ea', '#ec4899'],
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              display: false,
            },
          },
        },
      });
    },
    updateChart() {
      if (this.chart) {
        this
        
        
        .chart.data.datasets[0].data = [this.stock, this.demand, this.production || 0];
        this.chart.update();
      }
    },
    resetForm() {
      this.stock = 0;
      this.demand = 0;
      this.production = null;
      this.updateChart();
    },
  },
  mounted() {
    this.initializeChart();
  },
};
</script>

<style scoped>
body {
  font-family: 'Inter', sans-serif;
}
</style>
