<template>
  <div class="prefectures-area">
    <div v-for="prefecture in prefectures" :key="prefecture.id" class="prefecture">
      <label :for="prefecture.id">
        <input
          type="checkbox"
          :id="prefecture.id"
          :checked="prefecture.isChecked"
          @click="switchChart(prefecture.id, prefecture.name, prefecture.isChecked)"
        />
        {{ prefecture.name }}
      </label>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const ACCESS_TOKEN = process.env.VUE_APP_ACCESS_TOKEN;

export default {
  data() {
    return {
      prefectures: []
    };
  },
  mounted() {
    this.initPrefectures();
  },
  methods: {
    /* APIにアクセス */
    fetchAPI: function(path) {
      const response = axios.get(
        `https://opendata.resas-portal.go.jp/api/v1/${path}`,
        {
          headers: { "X-API-KEY": ACCESS_TOKEN }
        }
      );
      return response;
    },

    /* 県の初期表示 */
    initPrefectures: async function() {
      const path = "prefectures";
      try {
        const response = await this.fetchAPI(path);
        this.prefectures = response.data.result.map(val => {
          return {
            id: val["prefCode"],
            name: val["prefName"],
            isChecked: false
          };
        });
      } catch (error) {
        console.error(error.message);
      }
    },

    /* グラフを描画 */
    drawChart: async function(id, name) {
      const path = `population/composition/perYear?cityCode=-&prefCode=${id}`;
      try {
        const response = await this.fetchAPI(path);
        const population = response.data.result.data[0].data.map(
          val => val["value"]
        );
        this.$emit("onAddSeries", id, name, population);
        this.prefectures[id - 1].isChecked = true;
      } catch (error) {
        console.error(error.message);
      }
    },

    /* グラフを削除 */
    deleteChart: function(id) {
      this.$emit("onRemoveSeries", id);
      this.prefectures[id - 1].isChecked = false;
    },

    /* グラフの表示非表示を切り替え */
    switchChart: function(id, name, isChecked) {
      if (isChecked) {
        this.deleteChart(id);
      } else {
        this.drawChart(id, name);
      }
    }
  }
};
</script>
<style scoped>
.prefectures-area {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}

.prefectures {
  font-size: 15px;
}

label {
  cursor: pointer;
}

@media screen and (max-width: 425px) {
  .prefectures {
    font-size: 13px;
  }
}
</style>