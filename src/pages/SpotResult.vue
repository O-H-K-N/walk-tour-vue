<template>
  <v-container class="mb-16">
    <h2 class="mb-1 d-flex align-center justify-center">
      <v-icon left bottom color="cyan darken-1">mdi-earth</v-icon>
      <template v-if="this.$i18n.locale === 'ja'">
        {{ area.name }}
      </template>
      <template v-if="this.$i18n.locale === 'en'">
        {{ area.name_ens }}
      </template>
    </h2>
    <v-divider class="mb-2" style="max-width: 700px; margin: auto" />
    <v-row no-gutters class="my-5">
      <v-col>
        <GoogleMap
          :area="area"
          :spots="spots"
          :spot="spot"
          @get-video="getVideo"
          @click-count="clickCount"
        />
      </v-col>
    </v-row>

    <template v-if="videos.length != 0">
      <v-row style="max-width: 1200px; margin: auto">
        <v-col
          cols="12"
          sm="6"
          md="6"
          lg="6"
          class="d-lg-flex align-lg-end d-md-flex align-md-end d-sm-flex align-sm-end pb-0"
        >
          <h3 class="mb-2 d-flex align-center justify-center">
            <v-icon left bottom>mdi-map-marker</v-icon>
            <template v-if="this.$i18n.locale === 'ja'">
              {{ spotName }}
            </template>
            <template v-if="this.$i18n.locale === 'en'">
              {{ spotNameEns }}
            </template>
            <!--  ログイン中のユーザーにのみお気に入りマークを表示 -->
            <template v-if="authUser">
              <v-icon
                v-if="heart"
                right
                bottom
                class="d-flex d-sm-none"
                color="pink"
                @click="unBookmark"
                >mdi-heart</v-icon
              >
              <v-icon v-else right bottom class="d-flex d-sm-none" color="pink" @click="bookmark"
                >mdi-heart-outline</v-icon
              >
            </template>
          </h3>
          <!--  ログイン中のユーザーにのみお気に入りマークを表示 -->
          <template v-if="authUser">
            <v-icon
              v-if="heart"
              right
              bottom
              class="mb-2 d-none d-sm-flex"
              color="pink"
              @click="unBookmark"
              >mdi-heart</v-icon
            >
            <v-icon
              v-else
              right
              bottom
              class="mb-2 d-none d-sm-flex"
              color="pink"
              @click="bookmark"
              >mdi-heart-outline</v-icon
            >
          </template>
        </v-col>
        <v-col cols="12" sm="6" md="6" lg="6" class="pa-0">
          <v-tabs right v-model="currentTab">
            <v-tab @click="viewOrder(videos)" :value="'viewTab'">{{ $t('result.popular') }}</v-tab>
            <v-tab @click="newOrder(videos)">{{ $t('result.new') }}</v-tab>
          </v-tabs>
        </v-col>
      </v-row>

      <v-divider style="max-width: 1200px; margin: auto" />
    </template>
    <template v-else>
      <v-chip
        label
        large
        class="mt-10 d-flex justify-center"
        color="light-blue lighten-1"
        text-color="white"
        style="max-width: 700px; margin: auto"
      >
        <v-icon left>mdi-cursor-default-click-outline</v-icon>
        {{ $t('result.attention') }}
      </v-chip>
    </template>
    <Video :videos="videos" :area="area" :spot="spotName" :spotEns="spotNameEns" />
  </v-container>
</template>

<script>
import axios from '../plugins/axios';
import { mapGetters } from 'vuex';
import GoogleMap from '../components/TheGoogleMap.vue';
import Video from '../components/BaseVideo.vue';

export default {
  components: { GoogleMap, Video },
  metaInfo: {
    title: '検索結果',
  },
  props: {
    id: {
      type: [String, Number],
      required: true,
    },
    spotId: {
      type: [String, Number],
      required: false,
    },
  },
  data() {
    return {
      area: {},
      spots: [],
      spot: {},
      // お気に入り登録する際に使うspotデータ
      markSpot: {},
      heart: false,
      spotName: '',
      spotNameEns: '',
      videos: [],
      // タブのフォーカス
      currentTab: 'viewTab',
    };
  },
  created() {
    // spotIdを引き継いでいれば（ホットスポットからの遷移であれば）spotIdに関する動画も一覧表示する
    if (this.spotId === undefined) {
      this.setSpot();
    } else {
      this.setSpotAndVideo();
    }
  },
  computed: {
    // mapGettersでログイン中のユーザを取得
    ...mapGetters('users', ['authUser']),
  },
  methods: {
    // 国に関する地点を取得
    setSpot() {
      axios
        .get(`/countries/${this.id}`)
        .then((res) => {
          this.area = res.data.area;
          this.spots = this.area.spots;
        })
        .catch((error) => {
          console.log(error);
          this.$router.push({ name: 'Home' });
        });
    },
    // 地点に関する動画一覧ページに遷移する処理
    getVideo(spot) {
      // マーカークリックと同時に'閲覧順'にフォーカスを当てる
      this.currentTab = 'viewTab';
      axios
        .get(`spots/${spot.id}/videos`)
        .then((res) => {
          this.videos = res.data.videos;
          this.spotName = spot.name;
          this.spotNameEns = spot.name_ens;
          // お気に入り登録されているかを確認
          if (this.authUser) {
            this.bookmarked(spot);
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // 地点の表示と動画の一覧表示を同時に行う処理
    setSpotAndVideo() {
      axios
        .get(`/countries/${this.id}`)
        .then((res) => {
          this.area = res.data.area;
          this.spots = this.area.spots;
          let i = 0;
          // APIで取得した地点の中でpropsで引き継いだ地点と同じものを探す
          while (i < this.spots.length) {
            if (this.spots[i].id === this.spotId) {
              this.spot = this.spots[i];
              break;
            }
            i++;
          }
          // propsで引き継いだ地点の動画を取得
          this.getVideo(this.spot, this.area);
          // お気に入り登録されているかを確認
          if (this.authUser) {
            this.bookmarked(this.spot);
          }
        })
        .catch((error) => {
          console.log(error);
          this.$router.push({ name: 'Home' });
        });
    },
    // マーカー(地点)がクリックされた時に+1カウントされる
    clickCount(spot) {
      this.markSpot = spot;
      axios
        .get(`/spots/${spot.id}/edit`)
        .catch((error) => {
          console.log(error);
        });
    },
    // お気に入り済みかを確認
    bookmarked(spot) {
      axios
        .get(`/spots/${spot.id}`)
        .then((res) => {
          this.heart = res.data.spot.is_bookmarked
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // お気に入りに登録する
    bookmark() {
      this.heart = true;
      if (this.markSpot.id === undefined) {
        axios
          .post('/bookmarks', { id: this.spot.id })
          .catch((error) => {
            console.log(error);
          });
      } else {
        axios
          .post('/bookmarks', { id: this.markSpot.id })
          .catch((error) => {
            console.log(error);
          });
      }
    },
    // お気に入り登録を解除する
    unBookmark() {
      this.heart = false;
      if (this.markSpot.id === undefined) {
        axios
          .delete(`/bookmarks/${this.spot.id}`)
          .catch((error) => {
            console.log(error);
          });
      } else {
        axios
          .delete(`/bookmarks/${this.markSpot.id}`)
          .catch((error) => {
            console.log(error);
          });
      }
    },
    // 動画を新着順に並び替える
    newOrder(videos) {
      this.videos = videos.sort(function (a, b) {
        return a.published_at > b.published_at ? -1 : 1; //オブジェクトの昇順ソート
      });
    },
    // 動画を閲覧順に並び替える
    viewOrder(videos) {
      this.videos = videos
        .sort(function (a, b) {
          return a.view_count < b.view_count ? -1 : 1; //オブジェクトの昇順ソート
        })
        .reverse();
    },
  },
};
</script>
