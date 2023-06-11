---
title: "[FE構築3] ChartJS3をVue.jsアプリに導入し動的グラフ描画"
emoji: "📚"
type: "tech"
topics:
  - "vue3"
  - "chartjs"
  - "components"
published: true
published_at: "2023-04-21 15:41"
---

![](https://storage.googleapis.com/zenn-user-upload/5e9f4bd2d895-20230421.png)
# 概要
Vue.jsのアプリケーションにChartJSを使ってグラフを描画する。繋ぎ込みにvue-chartjsを利用する。ChartJS2はエラーが出たのでChartJS3を利用して描画する方法を採用する。

前の記事
https://zenn.dev/tofucode/articles/aeefc28c0766cd

# インストール
```
npm install chart.js vue-chartjs
```
# グラフ作成
components/BarChart.vue
```js
<template>
  <Bar :data="chartData" :options="chartOptions" />
</template>

<script>
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  name: 'BarChart',
  components: { Bar },
  props: {
    chartData: {
        type: Object,
        required: true
      },
    chartOptions: {
      type: Object,
      default: () => {}
    }
  }
}
</script>
```
main.vue（ユーザ入力をリアルタイムに反映したかったのでデータ作成をcomputedで行うが、データ直書きやfunctionなどでも良い。）
```js
<template>
   <BarChart :chartData="getGraphValue" :chartOptions="dat2" />
</tmplate>
<script>
import BarChart from './BarChart.vue';

export default {
  name: 'main page',
  components: {
    BarChart
  },
  data() {
    return {
      dat2: {
        responsive: true,
        animation: false,
      },
    }
  },
  computed: {
    getGraphValue: function () {
      return {
        labels: ['キー1', 'キー2','キー3','キー4','キー5'],
        datasets: [{
          label: 'グラフタイトル',
          data: [1, 2, 3, 4, 5],
          backgroundColor: ['#ccc','#ccc','#ccc','#ccc','#ccc']
        }]
      }
    }
  }
}
``` 

# 参照
https://vue-chartjs.org/ja/guide/
https://www.chartjs.org/docs/latest/developers/plugins.html

次の記事
https://zenn.dev/tofucode/articles/3009d8478dd084