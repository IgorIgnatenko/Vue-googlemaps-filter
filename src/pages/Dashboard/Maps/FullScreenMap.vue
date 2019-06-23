<template>
  <div id="app">
    <md-button class="md-success md-sm show__list" @click="showResults()">
      {{ this.btnMessage }} список
    </md-button>
    <div v-show="this.isLoading" ref="list" class="list">
      <p
        class="result__item"
        ref="result"
        v-for="(jres, index) in jsonResult"
        :key="jres.id"
      >
        <span class="result__id">{{ index + 1 }} </span>
        {{ jres.properties.name }}
      </p>
    </div>
    <yandex-map :coords="coords" :zoom="zoom" :controls="controls">
      <ymap-marker
        markerId="0"
        marker-type="placemark"
        hint-content="Метро Площадь Революции"
        :coords="[55.75645, 37.62335600000006]"
      >
      </ymap-marker>
      <ymap-marker
        v-for="(jcoords, index) in jsonResult"
        :key="jcoords.id"
        :markerId="index"
        marker-type="placemark"
        :hint-content="jcoords.properties.name"
        :coords="[
          jcoords.geometry.coordinates[1],
          jcoords.geometry.coordinates[0]
        ]"
      >
      </ymap-marker>
    </yandex-map>
  </div>
</template>

<script>
import { yandexMap, ymapMarker } from "vue-yandex-maps";

export default {
  name: "app",
  components: {
    yandexMap,
    ymapMarker
  },
  data: () => ({
    coords: [55.75645, 37.62335600000006],
    zoom: 13,
    controls: [],
    isLoading: false,
    btnMessage: "Показать",
    jsonResult: [
      {
        properties: {
          id: "",
          name: ""
        },
        geometry: {
          coordinates: []
        }
      }
    ]
  }),
  created() {
    this.initData();
  },
  methods: {
    initData() {
      let URLrequest = "https://search-maps.yandex.ru/v1/?";
      let params = {
        apikey: "2a8c3c2a-da28-4fc0-806d-0e91b034b8d6",
        lang: "ru_RU",
        results: 30,
        type: "biz",
        ll: "37.62335600000006,55.75645",
        spn: "0.03,0.03",
        rspn: true,
        text: "Музей"
      };
      for (let key in params) {
        URLrequest += `${key}=${params[key]}&`;
      }
      fetch(URLrequest)
        .then(response => {
          return response.json();
        })
        .then(data => {
          this.jsonResult = data.features;
          this.$refs.result.innerHTML = this.jsonResult;
        })
        .catch(error => {
          console.log(error.response);
        })
        .finally(() => {
          this.isLoading = true;
        });
    },
    showResults() {
      let btn = this.$refs.list;
      btn.classList.toggle("active");
      if (btn.classList.contains("active")) {
        this.btnMessage = "Скрыть";
      } else {
        this.btnMessage = "Показать";
      }
    }
  }
};
</script>

<style>
#app {
  position: relative;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding-top: 70px;
  display: -webkit-flex;
  display: -moz-flex;
  display: -ms-flex;
  display: -o-flex;
  display: flex;
}
.ymap-container {
  height: 100vh;
  width: 100%;
}
.list {
  position: absolute;
  z-index: 9;
  background-color: #fff;
  width: 0;
  overflow-y: scroll;
  max-height: 100vh;
  transition: 0.4s all ease;
}
.list.active {
  width: 25%;
}
.result__item {
  padding: 0 5px;
  margin: 5px 0;
  text-align: left;
  font-size: 14px;
  line-height: 1.4;
}
.result__item:not(:last-child) {
  border-bottom: 1px solid #ccc;
}
.result__id {
  font-weight: bold;
}
.show__list {
  position: absolute;
  top: 80px;
  right: 150px;
  z-index: 999;
}
@media (max-width: 768px) {
  .list.active {
    width: 35%;
  }
}
@media (max-width: 500px) {
  #app {
    display: flex;
    justify-content: center;
  }
  .list.active {
    top: 135px;
    width: 100%;
  }
  .show__list {
    right: inherit;
  }
}
</style>
