<template>
  <div>
    <BackGroundVideo :areas="areas" :spots="spots"></BackGroundVideo>
    <div class="mx-lg-16 my-8" style="color: #455A64;">
      <!-- 地点から動画を表示させるための検索窓 -->
      <!-- jqvmapで世界地図を表示 -->
      <WorldMap class="hidden-sm-and-down"></WorldMap>
      <!-- 最近新たに追加されたスポットを表示 -->
      <NewSpot class="mb-10"></NewSpot>
      <!-- ホットスポットの上位三つを表示 -->
      <HomeRanking></HomeRanking>
    </div>
  </div>
</template>

<script>
import WorldMap from '../components/WorldMap.vue'
import HomeRanking from '../components/HomeSpotRanking.vue'
import NewSpot from '../components/NewAddSpot.vue'
import BackGroundVideo from '../components/BackGroundVideo.vue'
import axios from '../plugins/axios'

export default {
  components: { WorldMap, HomeRanking, NewSpot, BackGroundVideo },
  data() {
    return {
      areas: [],
      spots: [],
    }
  },
  created() {
    this.getArea();
    this.getSpot();
  },
  methods: {
    // 地点を保有するエリア（国）を全て取得
    getArea() {
      axios.get('/countries')
      .then( res => {
        this.areas = res.data.areas;
      });
    },
    // 地点を全て取得
    getSpot(){
      axios.get('/spots', {
        params: {
          flag: 'all_spot'
        }
      })
      .then( res => {
        this.spots = res.data.spots;
      })
    }
  }
}
</script>