<script setup lang="ts">
import { ref, shallowRef, computed, watch, nextTick, Ref, ShallowRef } from 'vue' 
// ref = how we use state variable
// shallowRef = top-level ref (if we have an array, just looks at array, NOT its contents)
// computed = computing refs into other variables
// watch = check when something changes
// nextTick = wait until next tick to do something (delay)
import { Chart } from 'chart.js/auto'; // 'auto specification let's us use Chart.js quickly without importing other classes

import Weight from './types/weight'


const weights: Ref<Weight[]> = ref([])

const weightChartElement: Ref<HTMLCanvasElement | null> = ref(null)

const weightChart: ShallowRef<Chart | null> = shallowRef(null)  // this will be Chart.js obj, the shallowRef stops the reactive nature of the Chart obj.

const weightInput = ref(60.0)

const currentWeight = computed(() => {
  return weights.value.sort((a: Weight, b: Weight) => b.date - a.date)[0] || { weight: 0}
})

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })
}

watch(weights, newWeights => {
  const ws = [...newWeights]

  // if weightChart already exists, we can just re-sort data & update weightChart
  if (weightChart.value) {
    weightChart.value.data.labels = ws
      .sort((a, b) => a.date - b.date)
      .map(w => new Date(w.date).toLocaleDateString())
      .slice(-7)

    weightChart.value.data.datasets[0].data = ws
      .sort((a, b) => a.date - b.date)
      .map(w => w.weight)
      .slice(-7)

    weightChart.value.update()
    // call return to prevent nextTick() from being invoked. (Don't want to make entire new Chart obj)
    return
  }

  // nextTick() allows us to wait until there is data for the chart and that the canvas does exist
  // This is used to initialize the weightChart AFTER the first log has been added.
  nextTick(() => {
    if (weightChartElement.value) {
      weightChart.value = new Chart( weightChartElement.value, {
        type: 'line',
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: 'Weight',
              data: ws
                .sort((a, b) => a.date - b.date)
                .map(w => w.weight),
              backgroundColor: 'rgba(255, 105, 180, 0.2)',
              borderColor: 'rgb(255, 105, 180)',
              borderWidth: 1,
              fill: true
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false
        }
      })
    }
  })
  
}, {deep: true})  // need to set deep option to 'true' b/c weights is an array, we want to see when its contents update.
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <!-- Most Recent Log -->
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current Weight (kg)</small>
    </div>

    <!-- Input -->
    <!-- prevent default refresh from form submission -->
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Log weight" />
    </form>

    <!-- Info / Visualization -->
    <div class="data-container" v-if="weights && weights.length > 0">
      <!-- Chart -->
      <div class="chart-container">
        <h2>Last 7 Days</h2>
        <div class="canvas-box">
          <canvas id="weight-chart" ref="weightChartElement"></canvas>
        </div>
      </div>

      <!-- History List -->
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}kg</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>

  </main>
</template>

<style scoped>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
  }

  body {
    background-color: #eee;
  }

  main {
    padding: 1.5rem;
  }

  h1 {
    font-size: 2em;
    text-align: center;
    margin-bottom: 2rem;
  }

  h2 {
    margin-bottom: 1rem;
    color: #888;
    font-weight: 400;
  }

  .current {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    width: 200px;
    height: 200px;

    text-align: center;
    background-color: white;
    border-radius: 999px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border: 5px solid hotpink;

    margin: 0 auto 2rem;
  }

  .current span {
    display: block;
    font-size: 2em;
    font-weight: bold;
    margin-bottom: 0.5rem;
  }

  .current small {
    color: #888;
    font-style: italic;
  }

  form {
    display: flex;
    max-width: 720px;
    margin: 0 auto 2rem;
    border: 1px solid #aaa;
    border-radius: 0.5rem;
    overflow: hidden;
    transition: 200ms linear;
  }

  form:focus-within,
  form:hover {
    border-color: hotpink;
    border-width: 2px;
  }

  form input[type="number"] {
    appearance: none;
    outline: none;
    border: none;
    background-color: white;

    flex: 1 1 0%;
    padding: 1rem 1.5rem;
    font-size: 1.25rem;
  }

  form input[type="submit"] {
    appearance: none;
    outline: none;
    border: none;
    cursor: pointer;
    background-color: hotpink;

    padding: 0.5rem 1rem;

    color: white;
    font-size: 1.25rem;
    font-weight: 700;
    transition: 200ms linear;
    border-left: 3px solid transparent;
  }

  form input[type="submit"]:hover {
    background-color: white;
    color: hotpink;
    border-left-color: hotpink;
  }

  .chart-container {
    margin: 0 auto 2em;
    width: 100%;
    max-width: 720px;
  }

  .canvas-box {
    justify-content: center;
    background-color: white;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }

  .weight-history {
    max-width: 720px;
    margin: 0 auto;
  }

  .weight-history ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .weight-history ul li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem;
    cursor: pointer;
  }

  .weight-history ul li:nth-child(even) {
    background-color: #dfdfdf;
  }

  .weight-history ul li::hover {
    background-color: #f8f8f8;
  }

  .weight-history ul li:last-of-type {
    border-bottom: none;
  }

  .weight-history ul li span {
    display: block;
    font-size: 1.25rem;
    font-weight: 700;
    margin-right: 1rem;
  }

  .weight-history ul li small {
    color: #888;
    font-style: italic;
  }

</style>
