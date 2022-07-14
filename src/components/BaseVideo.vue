<template>
  <div>
    <v-row style="max-width: 1200px; margin: auto;">
      <v-col v-for="video in videos" :key="video.video_id" cols="12" sm="6" md="6" lg="4" class="my-1">
        <v-hover v-slot="{ hover }">
          <v-card :elevation="hover ? 12 : 2" max-width="400px" style="margin: auto;">
            <v-img :src="video.thumbnail" alt="サムネイル"  @click="openDialog(video);" style="cursor: pointer"></v-img>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>{{ video.title }}</v-list-item-title>
                <v-list-item-subtitle class="mt-1">{{ video.view_count.toLocaleString() }}回視聴・{{ video.published_at }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-card>
        </v-hover>
      </v-col>
    </v-row>

    <!-- ダイアログボックス -->
    <v-dialog v-model="dialog" max-width="1300px">
      <v-card>
        <v-col>
          <v-responsive :aspect-ratio="16/9">
            <iframe
              width="100%"
              height="100%"
              :src="urlForEmbedVideo"
              frameborder="0"
              autoplay
              allowfullscreen
              modestbranding="1"
            ></iframe>
          </v-responsive>
        </v-col>
        <v-card-subtitle class="py-0 font-weight-bold secondary--text">{{ title }}</v-card-subtitle>
        <v-card-subtitle class="my-0 pb-1 hidden-sm-and-down">{{ viewCount.toLocaleString() }}回視聴・{{ publishedAt }}</v-card-subtitle>
        <v-col class="d-flex justify-center pt-2">
          <v-btn
            color="blue darken-1"
            outlined
            style="text-transform: none"
            @click="shareTwitter(title, videoId)"
          >
            <v-icon left>mdi-twitter</v-icon>
            Twitterに共有
          </v-btn>
        </v-col>
        <v-divider class="mt-2"></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue darken-1"
            text
            @click="resetDialog()"
          >
            CLOSE
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  props: {
    videos: Array,
    area: String,
    spot: String,
  },
  data() {
    return {
      // ダイアログの表示、非表示
      dialog: false,
      title: '',
      videoId: '',
      viewCount: '',
      publishedAt: '',
      // Youtube動画の埋め込み用URL
      urlForEmbedVideo: ''
    }
  },
  // データの変更
  watch: {
    // ダイアログの状態を監視（表示、非表示の切り替えと同時にdataをリセット）
    dialog() {
      if (!this.dialog) {
        this.resetDialog()
      }
    },
  },
  methods: {
    // ダイアログの表示
    openDialog(video) {
      this.dialog = true;
      this.title = video.title;
      this.videoId = video.video_id;
      this.viewCount = video.view_count;
      this.publishedAt = video.published_at;
      this.urlForEmbedVideo = `https://www.youtube.com/embed/${this.videoId}`;
    },
    // ダイアログを非表示にしdataを空にする
    resetDialog() {
      this.dialog = false;
      this.title = '';
      this.videoId = '';
      this.description = '';
      this.viewCount = '';
      this.urlForEmbedVideo = '';
    },
    // VideoリンクをTwitterにシェア
    shareTwitter(title, videoId) {
      var shareURL =
      'https://twitter.com/intent/tweet?text='
      +
      `【${this.spot}（${this.area}）】`
      +
      '%0a'
      +
      '%0a'
      +
      encodeURI(title)
      +
      '%0a'
      +
      '&url='+ `https://youtu.be/${videoId}`+ '%20%40Youtube%20より'
      +
      '%0a'
      +
      '%23VtourHub';
      window.open(shareURL, '_blank');
    },
  }
}
</script>