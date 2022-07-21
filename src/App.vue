<template>
  <div id="app">
    <img src="./assets/graphic.png" alt="">
    <div class="weather">
      <section class="header">
        <div class="date">
          {{currentTime}}
        </div>
        <div class="city" @click="changeCity">
          <div class="current-city" v-if="city">
            {{ city }}, Slovakia 
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512"><path d="M168.3 499.2C116.1 435 0 279.4 0 192C0 85.96 85.96 0 192 0C298 0 384 85.96 384 192C384 279.4 267 435 215.7 499.2C203.4 514.5 180.6 514.5 168.3 499.2H168.3zM192 256C227.3 256 256 227.3 256 192C256 156.7 227.3 128 192 128C156.7 128 128 156.7 128 192C128 227.3 156.7 256 192 256z"/></svg>
          </div>
          <div class="current-city" v-else>
            Košice, Slovakia 
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512"><path d="M168.3 499.2C116.1 435 0 279.4 0 192C0 85.96 85.96 0 192 0C298 0 384 85.96 384 192C384 279.4 267 435 215.7 499.2C203.4 514.5 180.6 514.5 168.3 499.2H168.3zM192 256C227.3 256 256 227.3 256 192C256 156.7 227.3 128 192 128C156.7 128 128 156.7 128 192C128 227.3 156.7 256 192 256z"/></svg>
          </div>
        </div>
      </section>
      <section class="data">
        <div class="flex-row">
          <div class="flex-item">
            <img src="./assets/icons/weather.svg" alt="">
            <h3>
              {{ weather.weather[0].main }}
            </h3>
          </div>
          <div class="flex-item">
            <h2>
              {{ Math.round(weather.main.temp) }}
              <sup>°C</sup> 
            </h2>
          </div>
          <div class="flex-item">
            <h4>{{ Math.round(weather.main.temp_max) }} °C <img src="./assets/icons/up.svg" alt=""></h4>
            <h4>{{ Math.round(weather.main.temp_min) }} °C <img src="./assets/icons/down.svg" alt=""></h4>
          </div>
        </div>
        <div class="flex-row">
          <div class="flex-item">
            <img src="./assets/icons/humidity.svg" alt="">
            <h3>
              {{ weather.main.humidity }} %
            </h3>
            <p>Humidity</p>
          </div>
          <div class="flex-item">
            <img src="./assets/icons/bar.svg" alt="">
            <h3>
              {{ Math.round(weather.main.pressure) }} mBar
            </h3>
            <p>Pressure</p>
          </div>
          <div class="flex-item">
            <img src="./assets/icons/wind.svg" alt="">
            <h3>
              {{ Math.round(weather.wind.speed) }}km/h
            </h3>
            <p>Speed</p>
          </div>
        </div>
        <div class="flex-row">
          <div class="flex-item">
            <img src="./assets/icons/sunrise.svg" alt="">
            <h3>
              {{ formatDate(weather.sys.sunrise) }} AM
            </h3>
            <p>Sunrise</p>
          </div>
          <div class="flex-item">
            <img src="./assets/icons/sunset.svg" alt="">
            <h3>
              {{ formatDate(weather.sys.sunset) }} PM
            </h3>
            <p>Sunset</p>
          </div>
          <div class="flex-item">
            <img src="./assets/icons/date.svg" alt="">
            <h3>
              {{ formatDayTime(weather.sys.sunrise - weather.sys.sunset)  }}
            </h3>
            <p>Daytime</p>
          </div>
        </div>

        <div class="flex-row">
          <div class="flex-item forecast">
            <img src="./assets/icons/sun.svg" alt="">
            <h3>
              Thu, 09
            </h3>
            <p>35°C <img src="./assets/icons/up.svg" alt=""> 26°C <img src="./assets/icons/down.svg" alt=""></p>
          </div>
          <div class="flex-item forecast">
            <img src="./assets/icons/cloud.svg" alt="">
            <h3>
              Fri, 10
            </h3>
            <p>35°C <img src="./assets/icons/up.svg" alt=""> 27°C <img src="./assets/icons/down.svg" alt=""></p>
          </div>
          <div class="flex-item forecast">
            <img src="./assets/icons/hot.svg" alt="">
            <h3>
              Sat, 11
            </h3>
            <p>35°C <img src="./assets/icons/up.svg" alt=""> 29°C <img src="./assets/icons/down.svg" alt=""></p>
          </div>
        </div>
      </section>
    </div>
    <section class="modal" v-if="modal">
      <h2>Location</h2>
      <div class="search">
        <input type="text" name="Search" id="search" v-model="searchQuery" placeholder="Search city...">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512"><path d="M168.3 499.2C116.1 435 0 279.4 0 192C0 85.96 85.96 0 192 0C298 0 384 85.96 384 192C384 279.4 267 435 215.7 499.2C203.4 514.5 180.6 514.5 168.3 499.2H168.3zM192 256C227.3 256 256 227.3 256 192C256 156.7 227.3 128 192 128C156.7 128 128 156.7 128 192C128 227.3 156.7 256 192 256z"/></svg>
      </div>
      <ul>
        <li v-for="town in filteredItems" :key="town.id" @click="getWeather">
          <div>
            <div style="position:absolute; width:85%">
              {{ town.name }}
            </div>
          </div>
          <span>
            {{ town.temp }} °C
          </span>
        </li>
      </ul>
      <div v-if="err">
        City does not exist on Weather app API
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment';

export default {
  name: 'Weather-app',
  data() {
    return {
      weather: [], // push API to array
      modal: false, // open modal
      towns: [], // push cities to array
      searchQuery: '', // filter
      err: false, // show error message
      city: '', // name of the city with special characters
      currentTime: null, // show date and time
    }
  },
  computed: {
    // filter function
    filteredItems() {
      return this.towns.filter(town => {
        return town.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      })
    }
  },
  created () {
    this.currentTime = moment().locale('gb').format("dddd, D MMM YYYY | h:mA") // get current time using moment.js
    setInterval(() => this.updateCurrentTime(), 1000); // update time with setinterval

    // get API using axios
    axios.get('http://api.openweathermap.org/data/2.5/weather?q=kosice,sk&units=metric&APPID=f3c8d7fc899a6f96230607bf73d87cc7').then(response => {
      this.weather = response.data
    }).catch(error => {
      alert('Some shit is happen')
    })
    // get towns
    axios.get('towns.json').then(response => {
      this.towns = response.data
    }).catch(error => {
      alert('Some shit is happen')
    })
  },
  methods: {
    //show modal
    changeCity() {
      this.modal = true
    },

    //change weather by click
    getWeather(event) {
      let cityName = event.target.outerText.normalize("NFD").replace(/[\u0300-\u036f]/g, "") // get name of the city without special characters for API link building

      // get name of the city with special characters
      let name = event.target.outerText
      this.city  = name

      //API get request
      axios.get(`http://api.openweathermap.org/data/2.5/weather?q=${cityName},sk&units=metric&APPID=f3c8d7fc899a6f96230607bf73d87cc7`).then(response => {
        this.weather = response.data
        this.modal = false // close modal after click
      }).catch(error => {
        this.err = true // show error message
      })
    },

    // update current time
    updateCurrentTime() {
      this.currentTime = moment().locale('gb').format("dddd, D MMM YYYY | h:mA")
    },
    // format date time 
    formatDate(dateTime){ 
      return moment(dateTime).locale('sk').format('h:m');
    },
    //format Daytime 
    formatDayTime(dateTime){ 
      return moment(dateTime).locale('sk').format('h[h]m[m]');
    }
  },
}
</script>

<style lang="scss">
body{
  margin: 0;
  padding: 0;
  font-family: Barlow, sans-serif;
  font-size: 14px;
  color: #999999;
}
#app{
  img{
    width: 100%;
    height: auto;
    position: relative;
  }
  .weather{
    margin-top: -30px;
    background: #fff;
    position: relative;
    z-index: 10;
    border-radius: 27px;
    .header{
      display: flex;
      justify-content: space-between;
      .date{
        padding: 18px 0 0 5px;
      }
      .city{
        background: rgba(13, 159, 234, 0.08);
        color: #0DA0EA;
        padding: 15px 35px 14px 6px;
        border-radius: 0px 27px 0px 27px;
        .current-city {
          display: flex;
          align-items: center;
          svg{
            width: 8.27px;
            height: 12px;
            margin-left: 6.87px;
            fill: #0DA0EA;
          }
        }
      }
    }
    .data{
      .flex-row{
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 40px;
        margin-top: 24px;
        padding: 0 16px;
        .flex-item{
          text-align: center;
          width: 33.33%;
          img{
            width: 40px;
            height: 29.57px;
            margin-bottom: 16px;
          }
          h3,h4,h2{
            margin-bottom: 3px;
            margin-top: 0px;
          }
          h3{
            font-size: 16px;
            font-weight: 500;
            color: #000;
          }
          h2{
            font-size: 64px;
            font-weight: 300;
            color: #000;
            margin-left: 25px;
            sup{
              font-size: 24px;
              font-weight: 500;
              color: #666666;
              margin-left: -20px;
              position: relative;
              top: -12px;
            }
          }
          h4{
            font-size: 16px;
            font-weight: 300;
            color: #666666;
            img{
              width: 5px;
              height: 8px;
            }
            &:first-child{
              img{
                position: relative;
                top: -3px
              }
            }
          }
        }
        .forecast{
          box-shadow: 0px 8px 24px rgba(0, 0, 0, 0.1);
          border-radius: 16px;
          padding: 16px 18px;
          width: 20%;
          p{
            img{
              width: 5px;
              height: 7px;
              margin-bottom: 0px;
            }
          }
        }
      }
    }
  }
  .modal{
    position: fixed;
    top: 30px;
    left: 50%;
    transform: translateX(-50%);
    background: #fff;
    width: 100%;
    height: 140vh;
    z-index: 20;
    text-align: center;
    border-radius: 27px;
    h2{
      font-size: 16px;
      font-weight: 500;
      padding-top: 28px;
      margin-top: 0;
      margin-bottom: 27px;
    }
    .search{
      position: relative;
      margin-bottom: 34px;
      input{
        width: 85%;
        height: 40px;
        background: #F3F3F3;
        border-radius: 4px;
        border: none;
        outline: none;
        padding-left: 15px;
      }
      svg{
        position: absolute;
        top: 50%;
        right: 8%;
        transform: translateY(-50%);
        width: 11px;
        height: 16px;
      }
    }
    ul{
      list-style: none;
      padding: 0;
      margin: 0px 35px;
      li{
        display: flex;
        justify-content: space-between;
        margin-bottom: 10px;
        color: #444444;
        font-size: 18px;
        font-weight: 400;
        text-align: left;
      }
    }
  }
}
</style>
