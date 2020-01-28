<template>
  <div id="app">
    <h1>都道府県別の総人口推移グラフ</h1>
    <div class="prefecture-container">
      <h2>都道府県</h2>
      <div class="prefecture-area">
        <div v-for="prefecture in prefectures" :key="prefecture.id" class="prefecture">
          <label :for="prefecture.id">
            <input
              type="checkbox"
              :id="prefecture.id"
              :checked="prefecture.isChecked"
              @click="toggleChart(prefecture.id, prefecture.name, prefecture.isChecked)"
            />
            {{ prefecture.name }}
          </label>
        </div>
      </div>
    </div>
    <highcharts :options="options"></highcharts>
  </div>
</template>

<script>
import axios from "axios";
import { Chart } from "highcharts-vue";

const ACCESS_TOKEN = process.env.VUE_APP_ACCESS_TOKEN;

export default {
  name: "app",
  components: {
    highcharts: Chart
  },
  data() {
    return {
      prefectures: [],
      options: {
        title: {
          style: {
            display: "none"
          }
        },
        legend: {
          align: "right",
          verticalAlign: "top",
          layout: "vertical"
        },
        xAxis: {
          title: {
            text: "年度"
          },
          categories: [
            1960,
            1965,
            1970,
            1975,
            1980,
            1985,
            1990,
            1995,
            2000,
            2005,
            2010,
            2015,
            2020,
            2025,
            2030,
            2035,
            2040,
            2045
          ]
        },
        yAxis: {
          title: {
            text: "人口数"
          }
        },
        series: []
      }
    };
  },
  methods: {
    initPrefectures: function() {
      axios
        .get("https://opendata.resas-portal.go.jp/api/v1/prefectures", {
          headers: { "X-API-KEY": ACCESS_TOKEN }
        })
        .then(response => {
          this.prefectures = response.data.result.map(val => {
            return {
              id: val["prefCode"],
              name: val["prefName"],
              isChecked: false
            };
          });
        });
    },
    showChart: function(id, name) {
      axios
        .get(
          `https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?cityCode=-&prefCode=${id}`,
          {
            headers: {
              "X-API-KEY": ACCESS_TOKEN
            }
          }
        )
        .then(response => {
          const population = response.data.result.data[0].data.map(
            val => val["value"]
          );
          this.options.series.push({
            id: id,
            name: name,
            data: population
          });
          this.prefectures[id - 1].isChecked = true;
        });
    },
    hideChart: function(id) {
      this.options.series = this.options.series.filter(val => val.id !== id);
      this.prefectures[id - 1].isChecked = false;
    },
    toggleChart: function(id, name, isChecked) {
      if (isChecked) {
        this.hideChart(id);
      } else {
        this.showChart(id, name);
      }
    }
  },
  mounted() {
    this.initPrefectures();
  }
};
</script>
<style>
#app {
  max-width: 1024px;
  margin: 0 auto;
}

h1 {
  text-align: center;
  font-size: 20px;
}

.prefecture-container {
  margin-left: 4%;
}

h2 {
  font-size: 17px;
}

.prefecture-area {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}

.prefecture {
  font-size: 15px;
}

label {
  cursor: pointer;
}

@media screen and (max-width: 425px) {
  h1 {
    font-size: 18px;
  }

  h2 {
    font-size: 15px;
  }

  .prefecture-container {
    margin-left: 0;
  }

  .prefecture {
    font-size: 13px;
  }
}
</style>