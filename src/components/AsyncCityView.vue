<template>
  <section class="flex flex-col flex-1 items-center">
    <!--Banner-->
    <section
      v-if="route.query.preview"
      class="text-white p-4 w-full text-center bg-weather-secondary"
    >
      <p>
        You are currently previewing {{ route.params.city }}, click the "+" icon
        to start tracking this city
      </p>
    </section>
    <!--Weather Overview-->
    <section class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long"
          })
        }},
        {{
          new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
            timeStyle: "short"
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{
          Math.floor((Math.round(weatherData.current.temp) - 32) / 1.8)
        }}&deg;C
      </p>
      <div class="text-center">
        <p>
          Feels like
          {{
            Math.ceil((Math.round(weatherData.current.feels_like) - 32) / 1.8)
          }}&deg;C
        </p>
        <p class="capitalize">
          {{ weatherData.current.weather[0].description }}
        </p>
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
          :alt="weatherData.current.weather[0].description"
          class="w-[150px] h-auto"
        />
      </div>
    </section>

    <hr class="border-white border-opacity-10 border w-full" />
    <!--Hourly weather-->
    <section class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4 text-xl">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                  hour: "numeric"
                })
              }}
            </p>
            <img
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt="hourData.weather[0].description"
              class="w-auto h-[50px] object-cover"
            />
            <p class="text-xl text-white">
              {{ Math.round((hourData.temp - 32) / 1.8) }}&deg;C
            </p>
          </div>
        </div>
      </div>
    </section>

    <hr class="border-white border border-opacity-10 w-full" />

    <!--Weekly Weather-->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="text-xl mb-4">7-day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "long"
              })
            }}
          </p>
          <img
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            :alt="day.weather[0].description"
            class="w-[50px] h-[50px] object-cover"
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round((day.temp.max - 32) / 1.8) }}&deg;C</p>

            <p>L: {{ Math.round((day.temp.min - 32) / 1.8) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove city</p>
    </div>
  </section>
</template>

<script setup>
import axios from "axios"
import { useRoute, useRouter } from "vue-router"

const route = useRoute()

const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    )

    const localOffset = new Date().getTimezoneOffset() * 60000
    const utc = weatherData.data.current.dt * 1000 + localOffset
    weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset
    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset
    })
    return weatherData.data
  } catch (err) {
    console.log(err)
  }
}
const weatherData = await getWeatherData()

const router = useRouter()
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"))

  const updatedCities = cities.filter((city) => city.id !== route.query.id)

  localStorage.setItem("savedCities", JSON.stringify(updatedCities))

  router.push({
    name: "home"
  })
}
</script>
