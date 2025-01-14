<template>
  <div
    class="min-h-screen bg-gray-100 flex flex-col items-center justify-center"
  >
    <div
      class="bg-white shadow-2xl rounded-2xl p-10 max-w-lg w-full transform transition-transform hover:scale-105"
    >
      <h1 class="text-5xl font-extrabold text-center mb-8 text-indigo-600">
        Sistem Fuzzy Produksi
      </h1>
      <div class="flex flex-col gap-6">
        <div>
          <label
            for="demand"
            class="block text-lg font-medium text-gray-700 mb-2"
            >Jumlah Permintaan:</label
          >
          <input
            type="number"
            v-model="demand"
            id="demand"
            class="border border-indigo-300 rounded-lg p-3 w-full focus:ring focus:ring-indigo-400 focus:border-indigo-500 shadow-sm"
          />
        </div>
        <div>
          <label
            for="stock"
            class="block text-lg font-medium text-gray-700 mb-2"
            >Jumlah Persediaan:</label
          >
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
          @click="resetFields"
          class="bg-gray-500 text-white font-bold py-3 rounded-lg w-full hover:bg-gray-600 transition-transform transform hover:scale-105 shadow-md"
        >
          Reset
        </button>
      </div>
      <div v-if="production !== null" class="mt-8 text-center">
        <h2 class="text-2xl font-semibold text-gray-800">
          Produksi yang Disarankan:
          <span class="text-indigo-600 text-3xl font-bold">{{
            production
          }}</span>
        </h2>
      </div>
    </div>
    <div
      v-if="fuzzificationData && production !== null"
      class="bg-white shadow-lg rounded-lg p-6 max-w-4xl w-full mt-8"
    >
      <h1 class="text-3xl font-bold text-center text-indigo-600 mb-6">
        Chart Fuzzifikasi
      </h1>
      <div class="">
        <div>
          <h2 class="text-xl font-semibold text-gray-700 mb-4">
            Fuzzifikasi Permintaan
          </h2>
          <canvas id="demandChart"></canvas>
        </div>
        <div>
          <h2 class="text-xl font-semibold text-gray-700 mb-4">
            Fuzzifikasi Persediaan
          </h2>
          <canvas id="stockChart"></canvas>
        </div>
        <div>
          <h2 class="text-xl font-semibold text-gray-700 mb-4">
            Fuzzifikasi Produksi
          </h2>
          <canvas id="productionChart"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from "chart.js";
Chart.register(...registerables);

export default {
  data() {
    return {
      demand: 0,
      stock: 0,
      production: null,
      fuzzificationData: null,
      demandChart: null,
      stockChart: null,
      productionChart: null,
      himPersediaan: null,
      himPermintaan: null,
    };
  },
  methods: {
    async fetchFuzzificationData() {
      try {
        const response = await fetch("http://localhost:3000/fuzzification");
        const data = await response.json();
        this.fuzzificationData = data;

        // Buat chart setelah data fuzzifikasi tersedia
        this.createDemandChart();
        this.createStockChart();
        this.createProductionChart();
      } catch (error) {
        console.error("Error fetching fuzzification data:", error);
      }
    },
    resetFields() {
      this.demand = 0;
      this.stock = 0;
      this.production = null;
      if (this.demandChart) {
        this.demandChart.destroy();
        this.demandChart = null;
      }
      if (this.stockChart) {
        this.stockChart.destroy();
        this.stockChart = null;
      }
      if (this.productionChart) {
        this.productionChart.destroy();
        this.productionChart = null;
      }
    },
    async calculateProduction() {
      try {
        const response = await fetch("http://localhost:3000/prediksi", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            permintaan: this.demand,
            persediaan: this.stock,
          }),
        });
        const data = await response.json();
        this.production = data.produksi;
        this.himPermintaan = data.derajat_keanggotaan.him_permintaan;
        this.himPersediaan = data.derajat_keanggotaan.him_persediaan;

        // Ambil data fuzzifikasi setelah perhitungan
        await this.fetchFuzzificationData();
      } catch (error) {
        console.error("Error:", error);
      }
    },

    createDemandChart() {
      const ctx = document.getElementById("demandChart").getContext("2d");
      const { range, rendah, sedang, banyak } =
        this.fuzzificationData.permintaan;

      // Nilai derajat keanggotaan yang diambil dari data produksi
      const himPermintaan = this.himPermintaan;
      console.log(himPermintaan);

      this.demandChart = new Chart(ctx, {
        type: "line",
        data: {
          labels: range,
          datasets: [
            {
              label: "Permintaan Rendah",
              data: rendah,
              borderColor: "#ef4444",
              backgroundColor: "rgba(239, 68, 68, 0.2)",
              fill: true,
            },
            {
              label: "Permintaan Sedang",
              data: sedang,
              borderColor: "#3b82f6",
              backgroundColor: "rgba(59, 130, 246, 0.2)",
              fill: true,
            },
            {
              label: "Permintaan Banyak",
              data: banyak,
              borderColor: "#10b981",
              backgroundColor: "rgba(16, 185, 129, 0.2)",
              fill: true,
            },
            // Tambahkan dataset untuk nilai derajat keanggotaan
            {
              label: "Derajat Keanggotaan Rendah",
              data: Array(range.length).fill(himPermintaan.rendah),
              borderColor: "#991b1b",
              borderDash: [5, 5],
              pointStyle: "rectRot",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Derajat Keanggotaan Sedang",
              data: Array(range.length).fill(himPermintaan.sedang),
              borderColor: "#1e40af",
              borderDash: [5, 5],
              pointStyle: "triangle",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Derajat Keanggotaan Banyak",
              data: Array(range.length).fill(himPermintaan.banyak),
              borderColor: "#065f46",
              borderDash: [5, 5],
              pointStyle: "circle",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Input Permintaan",
              data: range.map((x) => (x === this.demand ? 1 : 0)),
              borderColor: "#f97316",
              borderDash: [5, 5],
              pointStyle: "rect",
              pointRadius: 6,
              fill: false,
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: "top",
            },
          },
          scales: {
            x: {
              title: {
                display: true,
                text: "Jumlah Permintaan",
              },
            },
            y: {
              title: {
                display: true,
                text: "Derajat Keanggotaan",
              },
              min: 0,
              max: 1,
            },
          },
        },
      });
    },

    createStockChart() {
      const ctx = document.getElementById("stockChart").getContext("2d");
      const { range, minim, sedang, banyak } =
        this.fuzzificationData.persediaan;
      const himPersediaan = this.himPersediaan;

      this.stockChart = new Chart(ctx, {
        type: "line",
        data: {
          labels: range,
          datasets: [
            {
              label: "Persediaan Minim",
              data: minim,
              borderColor: "#ef4444",
              backgroundColor: "rgba(239, 68, 68, 0.2)",
              fill: true,
            },
            {
              label: "Persediaan Sedang",
              data: sedang,
              borderColor: "#3b82f6",
              backgroundColor: "rgba(59, 130, 246, 0.2)",
              fill: true,
            },
            {
              label: "Persediaan Banyak",
              data: banyak,
              borderColor: "#10b981",
              backgroundColor: "rgba(16, 185, 129, 0.2)",
              fill: true,
            },
            {
              label: "Derajat Keanggotaan Rendah",
              data: Array(range.length).fill(himPersediaan.rendah),
              borderColor: "#991b1b",
              borderDash: [5, 5],
              pointStyle: "rectRot",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Derajat Keanggotaan Sedang",
              data: Array(range.length).fill(himPersediaan.sedang),
              borderColor: "#1e40af",
              borderDash: [5, 5],
              pointStyle: "triangle",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Derajat Keanggotaan Banyak",
              data: Array(range.length).fill(himPersediaan.banyak),
              borderColor: "#065f46",
              borderDash: [5, 5],
              pointStyle: "circle",
              pointRadius: 1,
              fill: false,
            },
            {
              label: "Input Persediaan",
              data: range.map((x) => (x === this.stock ? 1 : 0)),
              borderColor: "#f97316",
              borderDash: [5, 5],
              pointStyle: "rect",
              pointRadius: 6,
              fill: false,
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: "top",
            },
          },
          scales: {
            x: {
              title: {
                display: true,
                text: "Jumlah Persediaan",
              },
            },
            y: {
              title: {
                display: true,
                text: "Derajat Keanggotaan",
              },
              min: 0,
              max: 1,
            },
          },
        },
      });
    },
    createProductionChart() {
      const ctx = document.getElementById("productionChart").getContext("2d");
      const { range, kecil, sedang, besar } = this.fuzzificationData.produksi;

      this.productionChart = new Chart(ctx, {
        type: "line",
        data: {
          labels: range,
          datasets: [
            {
              label: "Produksi Kecil",
              data: kecil,
              borderColor: "#ef4444",
              backgroundColor: "rgba(239, 68, 68, 0.2)",
              fill: true,
            },
            {
              label: "Produksi Sedang",
              data: sedang,
              borderColor: "#3b82f6",
              backgroundColor: "rgba(59, 130, 246, 0.2)",
              fill: true,
            },
            {
              label: "Produksi Besar",
              data: besar,
              borderColor: "#10b981",
              backgroundColor: "rgba(16, 185, 129, 0.2)",
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: "top",
            },
          },
          scales: {
            x: {
              title: {
                display: true,
                text: "Jumlah Produksi",
              },
            },
            y: {
              title: {
                display: true,
                text: "Derajat Keanggotaan",
              },
              min: 0,
              max: 1,
            },
          },
        },
      });
    },
  },
  mounted() {
    this.fetchFuzzificationData();
  },
};
</script>

<style scoped>
body {
  font-family: "Inter", sans-serif;
}
</style>
