<template>
  <div class="wrapper">
    <header>
      <h1>Calculadora Price e SAC</h1>
    </header>

    <div class="form">
      <div class="form-group">
        <label for="valor">Valor Presente (R$)</label>
        <input id="valor" v-model.number="valor" type="number" placeholder="Ex: 10000" />
      </div>
      <div class="form-group">
        <label for="taxa">Taxa de Juros Mensal (%)</label>
        <input id="taxa" v-model.number="taxa" type="number" placeholder="Ex: 2" />
      </div>
      <div class="form-group">
        <label for="parcelas">Número de Meses</label>
        <input id="parcelas" v-model.number="parcelas" type="number" placeholder="Ex: 12" />
      </div>
      <div class="form-group">
        <label for="metodo">Método de Amortização</label>
        <select id="metodo" v-model="metodo">
          <option value="price">Price</option>
          <option value="sac">SAC</option>
        </select>
      </div>
      <button @click="calcular">Calcular</button>
    </div>

    <button @click="exportarPDF" class="exportar">Exportar PDF</button>
    <canvas id="grafico" height="200" style="margin-top: 20px;"></canvas>

    <table v-if="tabela.length">
      <thead>
        <tr>
          <th>Parcela</th>
          <th>Amortização</th>
          <th>Juros</th>
          <th>Parcela</th>
          <th>Saldo</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, i) in tabela" :key="i">
          <td>{{ i + 1 }}</td>
          <td>{{ item.amort.toFixed(2) }}</td>
          <td>{{ item.juros.toFixed(2) }}</td>
          <td>{{ item.total.toFixed(2) }}</td>
          <td>{{ item.saldo.toFixed(2) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'
import Chart from 'chart.js/auto'
import html2pdf from 'html2pdf.js'

const valor = ref(10000)
const taxa = ref(2)
const parcelas = ref(12)
const metodo = ref('price')
const tabela = ref([])

let chartInstance = null

const calcular = () => {
  tabela.value = []
  let saldo = valor.value
  const i = taxa.value / 100

  if (metodo.value === 'price') {
    const p = valor.value * (i * Math.pow(1 + i, parcelas.value)) / (Math.pow(1 + i, parcelas.value) - 1)
    for (let n = 0; n < parcelas.value; n++) {
      const juros = saldo * i
      const amort = p - juros
      saldo -= amort
      tabela.value.push({ amort, juros, total: p, saldo: saldo < 0 ? 0 : saldo })
    }
  } else {
    const amort = valor.value / parcelas.value
    for (let n = 0; n < parcelas.value; n++) {
      const juros = saldo * i
      const total = amort + juros
      saldo -= amort
      tabela.value.push({ amort, juros, total, saldo: saldo < 0 ? 0 : saldo })
    }
  }
  renderizarGrafico()
}

const renderizarGrafico = () => {
  nextTick(() => {
    const ctx = document.getElementById('grafico')
    if (ctx) {
      if (chartInstance) chartInstance.destroy()
      chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: tabela.value.map((_, i) => 'Parcela ' + (i + 1)),
          datasets: [
            {
              label: 'Amortização',
              data: tabela.value.map(item => item.amort),
              backgroundColor: '#42b983'
            },
            {
              label: 'Juros',
              data: tabela.value.map(item => item.juros),
              backgroundColor: '#ff6384'
            }
          ]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: 'top' },
            title: { display: true, text: 'Amortização x Juros por Parcela' }
          }
        }
      })
    }
  })
}

const exportarPDF = () => {
  html2pdf().set({ margin: 1, filename: 'simulacao-financiamento.pdf' }).from(document.body).save()
}
</script>

<style scoped>
.wrapper {
  max-width: 900px;
  margin: auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #333;
}
header {
  background: #0057b7;
  padding: 20px;
  color: white;
  text-align: center;
  border-radius: 8px;
  margin-bottom: 20px;
}
.form {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}
.form-group {
  display: flex;
  flex-direction: column;
}
label {
  margin-bottom: 6px;
  font-weight: 600;
}
input, select {
  padding: 10px;
  font-size: 16px;
  border-radius: 6px;
  border: 1px solid #ccc;
}
button {
  grid-column: span 2;
  padding: 12px;
  background: #0057b7;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
}
button:hover {
  background: #003d87;
}
.exportar {
  margin-top: 20px;
  background: #28a745;
}
.exportar:hover {
  background: #218838;
}
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}
th {
  background: #f1f1f1;
}
th, td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: center;
}
tbody tr:nth-child(even) {
  background-color: #fafafa;
}
</style>
