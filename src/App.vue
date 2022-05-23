<script>
import {ref, reactive, onMounted, computed} from 'vue';

export default {
  setup() {
    const country = ref(null);
    const unit = ref('Celsius');
    const key = '7160b9e28c61a6f6fcdf45a96a2734f8';
    const data = reactive({
      name: '-',
      temp: 0,
      feels_like: 0,
      humidity: '0',
      description: '-',
    })
    const tempUnits = {
      c: 'Celsius',
      f: 'Fahrenheit'
    }
    
    const isCelsius = computed(() => unit.value === tempUnits.c);
    const tempLetter = computed(() => unit.value === tempUnits.c ? '°C' : '°F');

    async function makeWeatherRequest(country = 'Montreal') {
      try {
        const units = isCelsius.value ? 'metric' : 'imperial';
        const response = await fetch(`http://api.openweathermap.org/data/2.5/weather?q=${country}&APPID=${key}&units=${units}`);
        const {name, main: {temp, feels_like, humidity}, weather: [{description}]} =  await response.json();
        data.name = name;
        data.temp = Math.round(temp);
        data.feels_like = Math.round(feels_like);
        data.description = description;
        data.humidity = humidity;
      } catch(e) {
        throw new Error(e);
      }
    }

    async function onEnter() {
      await makeWeatherRequest(country.value);
    }

    function onUpdateTemp() {
      const toF = v => Math.round((Number(v) * 1.8) + 32);
      const toC = v => Math.round((Number(v) - 32) * 0.5556); 
      unit.value = isCelsius.value ? tempUnits.f : tempUnits.c;
      data.temp = isCelsius.value ? toC(data.temp) : toF(data.temp);
      data.feels_like = isCelsius.value ? toC(data.feels_like) : toF(data.feels_like);
    }

    onMounted(async () => {
      await makeWeatherRequest();
    });

    return {
      country,
      data,
      unit,
      onEnter,
      onUpdateTemp,
      tempLetter
    }
  },
}
</script>

<template>
  <div class="card">
    <input placeholder="Search and press enter ↵" type="text" v-model="country" @keyup.enter="onEnter">
    <button @click="onUpdateTemp">{{unit}}</button>
    <p>{{data.name}}</p>
    <h1>{{data.temp}}{{tempLetter}}</h1>
    <p>{{data.description}}</p>
    <p><span>feels like</span> {{data.feels_like}}{{tempLetter}}</p>
    <p><span>humidity</span> {{data.humidity}}%</p>
  </div>

</template>

<style>
html {
  margin: 0;
  padding: 0;
}
body {
  height: 100vh;
  background: lightsalmon;
  padding: 0;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  line-height: 1.5;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 18px;
  text-align: center;
  color: #2c3e50;
}
.card {
  width: 300px;
  box-sizing: border-box;
  padding: 26px;
  background: white;
  box-shadow: 0 0 0 6px #2c3e50, -16px 16px 0 0 #2c3e50;
}
.card * + * {
  margin: 0;
}
.card button {
  border: 2px solid #2c3e50;
  background: lightgrey;
  padding: 12px 20px;
  font-weight: bolder;
  width: 100%;
  font-size: 18px;
  cursor: pointer;
  color: #2c3e50;
  margin-bottom: 24px;
  line-height: 1.5;
}
.card input {
  border: 2px solid #2c3e50;
  background: white;
  padding: 12px 12px;
  color: #2c3e50;
  box-sizing: border-box;
  font-weight: bolder;
  width: 100%;
  font-size: 18px;
  border-radius: 0;
  margin-bottom: 12px;
  line-height: 1.5;
}
.card h1 {
  font-size: 300%;
  line-height: 1;
  margin-bottom: 12px;
}
.card span {
  opacity: 0.6;
}
</style>
