<script setup>
import { ref, computed, onMounted } from 'vue';

const apiKey = ''
const weatherURL = 'https://api.openweathermap.org/data/2.5/weather'
const cityURL = 'http://api.openweathermap.org/geo/1.0/direct?'
let mode = "metric"
const tempUnit = ref("°C")
const selectedCity = ref("")
const weatherData = ref({

    cityName: "",
    countryName: "",
    weatherDescription: "",
    currentTemp: 0,
    minTemp: 0,
    maxTemp: 0, 
    feelsLike: 0,
    humidity: 0,
    precipitation: 0

})

const searchBarIsEmpty = computed(() => {
  return selectedCity.value.trim().length === 0 ? true : false
})

async function weatherRequest(coordinates){

    //hit weather API using coordinates passed as parameters
    
    try {
        const apiResponse = await fetch(`${weatherURL}?lat=${coordinates.latitude}&lon=${coordinates.longitude}&appid=${apiKey}&units=${mode}`)
        const apiData = await apiResponse.json()
        console.log(apiData)
    //weather description
        let weatherText = apiData["weather"][0]["description"]
        weatherData.value.weatherDescription = capitalizeFirstLetter(weatherText)
        //temperature 
        weatherData.value.currentTemp = Math.floor(apiData["main"]["temp"])  
        weatherData.value.feelsLike = Math.floor(apiData["main"]["feels_like"])
        //everything else
        weatherData.value.humidity = apiData["main"]["humidity"]
        weatherData.value.minTemp = apiData["main"]["temp_min"]
        weatherData.value.maxTempTemp = apiData["main"]["temp_max"]
    } catch (error) {
        console.log(error)
    }
    

}

async function cityRequest(){
    //hit city API for the city in search bar
    try {
        const apiResponse = await fetch(`${cityURL}q=${selectedCity.value.trim()}&limit=3&appid=${apiKey}`)
        const apiData = await apiResponse.json()
        if (apiData){
        //grab city and country code
        weatherData.value.cityName = apiData["0"]["name"]
        weatherData.value.countryName = apiData["0"]["country"]
        //use the coordinates of the first element in the JSON
        const coordinates = {
            latitude : apiData["0"]["lat"],
            longitude : apiData["0"]["lon"]
        }
         weatherRequest(coordinates)
        } else
            console.log("City not found")
        
    } catch (error) {
        console.log(error)
    }
    //if city is found, apiData should be populated
    
    

}

function capitalizeFirstLetter(string) {
    return string.charAt(0).toUpperCase() + string.slice(1);
}

function modeSwitch() {
    if (mode === "metric"){
        mode = "imperial"
        tempUnit.value = "°F"
        //change all temps to °F if city has already been selected (data is already on screen)
        if (weatherData.value.cityName){
            weatherData.value.currentTemp = Math.floor(convertToFahrenheit(weatherData.value.currentTemp))
            weatherData.value.feelsLike = Math.floor(convertToFahrenheit(weatherData.value.feelsLike))
        }
    } else if (mode === "imperial"){
        mode = "metric"
        tempUnit.value = "°C"
        //change all temps to °C if city has already been selected (data is already on screen)
        if (weatherData.value.cityName){
            weatherData.value.currentTemp = Math.floor(convertToCelsius(weatherData.value.currentTemp))
            weatherData.value.feelsLike = Math.floor(convertToCelsius(weatherData.value.feelsLike))
        }
    }
}

function convertToFahrenheit(temp){
    return (temp*1.8) + 32
}

function convertToCelsius(temp){
    return (temp-32)/1.8 
}

</script>

<template >
    <div class="search">
        <input v-model="selectedCity" class="search-bar" type="text" placeholder="Enter city name" title="Enter city name"/>
        <button id="search-btn" class="button" @click="cityRequest()" v-bind:disabled="searchBarIsEmpty" title="Search">
            <img id="search-icon" src="../assets/icons8-search-50.png"/>
        </button>
        <button id="toggle-units-btn" class="button" @click="modeSwitch()" title="Toggle imperial or metric units">
            {{ tempUnit }}
        </button>
    </div>
    <img src="../assets/23527-3-weather.png" width="250" height="250"/>
    <h1 class="city">Weather in <a id="city-name">{{ weatherData.cityName }}, {{ weatherData.countryName }}</a></h1>
    <div v-if="weatherData" class="conditions">
        <h1 id="temperature">{{ weatherData.currentTemp }} {{tempUnit}}</h1>
        <h3 id="feels-like">Feels like: {{ weatherData.feelsLike }} {{tempUnit}}</h3>
    </div>
    <h1 class="conditions" id="current-weather">{{weatherData.weatherDescription}}</h1>
        <div>
            <h2 id="humidity">Humidity: <h3>{{weatherData.humidity}}%</h3></h2>
        </div>

</template>

<style scoped>

.search-bar{
    height: 40px;
    margin: 10px;
    padding: 15px;
    border: none;
    border-radius: 25px;
    box-shadow: 2px 2px 1px rgb(80, 80, 80);
}

.button {
    width: 43px;
    height: 43px;
    margin: 10px;
    margin-left: 5px;
    border: none;
    border-radius: 50%;
    box-shadow: 2px 2px 1px rgb(80, 80, 80);
    
}
.conditions{
    display: grid;
}

.precipitation{
    margin-left: 10px;
}

#feels-like {
    font-size: small;
}

#search-icon{
    width: 16px;
    height: 16px;
}

#temperature{
    font-weight: bolder;
    padding-top: 20px;
    padding-bottom: 5px;
}
#current-weather{

    font-weight:500;

}
#humidity{

    margin-right: 10px;

}

#precipiation{

    margin-right: 10px;

}


</style>