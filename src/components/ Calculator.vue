<template>
  <div class="calculator">
    <h1>dB Calculator</h1>
    <div class="input-group">
      <label for="sensitivity">Sensitivity (dB/mW):</label>
      <input v-model.number="sensitivity" type="number" id="sensitivity" />
    </div>
    <div class="input-group">
      <label for="impedance">Impedance (Ω):</label>
      <input v-model.number="impedance" type="number" id="impedance" />
    </div>
    <div class="input-group">
      <label for="loudness">Loudness (dB):</label>
      <input v-model.number="loudness" type="number" id="loudness" />
    </div>
    <button @click="calculate">Calculate</button>
    <div v-if="results" class="results">
      <p>Required Power: {{ results.power.toFixed(2) }} mW</p>
      <p>Required Voltage: {{ results.voltage.toFixed(2) }} V RMS</p>
      <p>Required Current: {{ results.current.toFixed(2) }} mA RMS</p>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "Calculator",
  setup() {
    const sensitivity = ref<number>(98); // Default sensitivity in dB/mW
    const impedance = ref<number>(32); // Default impedance in Ω
    const loudness = ref<number>(110); // Default loudness in dB

    const results = ref<{ power: number; voltage: number; current: number } | null>(null);

    const calculate = () => {
      const power = Math.pow(10, (loudness.value - sensitivity.value) / 10);
      const voltage = Math.sqrt(power * impedance.value);
      const current = voltage / impedance.value * 1000;
      results.value = { power, voltage, current };
    };

    return {
      sensitivity,
      impedance,
      loudness,
      results,
      calculate,
    };
  },
});
</script>

<style scoped>
.calculator {
  max-width: 400px;
  margin: 0 auto;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  font-family: Arial, sans-serif;
}

.input-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input {
  width: 100%;
  padding: 0.5rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  display: block;
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.results {
  margin-top: 1rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}
</style>
