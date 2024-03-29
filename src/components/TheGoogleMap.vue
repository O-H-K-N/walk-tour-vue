<template>
  <div ref="map" class="map" />
</template>

<script>
// GoogleMapのマーカーをクラスタリング
import { MarkerClusterer } from '@googlemaps/markerclusterer';

export default {
  props: {
    area: {
      type: Object,
      required: true,
    },
    spots: {
      type: Array,
      required: true,
    },
    spot: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      latlng: [],
      // Youtube動画の埋め込み用URL
      videos: [],
      randomVideo: {},
      urlForEmbedVideo: '',
      // アクティブ中のマーカー
      focusMarker: null,
    };
  },
  // データの変更
  watch: {
    // ダイアログの状態を監視（表示、非表示の切り替えと同時にdataをリセット）
    dialog() {
      if (!this.dialog) {
        this.resetDialog();
      }
    },
  },
  mounted() {
    let timer = setInterval(() => {
      const google = window.google;
      if (google) {
        clearInterval(timer);

        // マーカー
        let marker = [];
        let marker_2 = [];
        let markers = [];
        // クリック時に表示される情報ウィンドウ
        const infoWindow = new google.maps.InfoWindow({
          content: '',
          disableAutoPan: true,
        });
        // カーソルホバー時に表示される情報ウィンドウ
        const infoWindowHover = new google.maps.InfoWindow({
          content: '',
          disableAutoPan: true,
        });

        const standardIcon = 'http://maps.google.co.jp/mapfiles/ms/icons/blue-dot.png';
        const focusIcon = 'http://maps.google.co.jp/mapfiles/ms/icons/yellow-dot.png';

        let map = new google.maps.Map(this.$refs.map, {
          // マップを全画面モードで表示するボタンを非表示
          fullscreenControl: false,
          // ストリートビューに切り替えるボタンを非表示
          streetViewControl: false,
          //地図と航空写真を切り替えるボタンを非表示
          mapTypeControl: false,
          scrollwheel: false,
          zoom: 4,
        });

        // エリア（国、都道府県）から緯度経度を取得し地図に表示
        new google.maps.Geocoder().geocode({ address: this.area.name }, (results, status) => {
          if (status === google.maps.GeocoderStatus.OK) {
            if (results[0].geometry) {
              // 緯度経度を取得
              this.latlng = results[0].geometry.location;
              // 地図の中心に配置
              map.setCenter(this.latlng);
              // 全エリアが表示されるように設定
              map.fitBounds(results[0].geometry.bounds);
            }
          } else if (status === google.maps.GeocoderStatus.ZERO_RESULTS) {
            alert('見つかりません');
          } else {
            // console.log(status);
            alert('エラー発生');
          }
        });

        // エリア内の各地点にマーカーを立てる
        for (let i = 0; i < this.spots.length; i++) {
          // spotIdを引き継いでおり（ホットスポットからの遷移であり）、 APIで取得した地点の中で同じものであれば
          if (this.spots[i] === this.spot) {
            marker = new google.maps.Marker({
              position: { lat: Number(this.spot.lat), lng: Number(this.spot.lng) },
              map,
              icon: focusIcon,
              animation: google.maps.Animation.BOUNCE,
            });
            markers.push(marker);

            // 情報ウィンドウを固定で表示する
            if (this.$i18n.locale === 'ja') {
              infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name}</h2></div>`);
            } else if (this.$i18n.locale === 'en') {
              infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name_ens}</h2></div>`);
            }
            infoWindow.open(map, marker);

            // マーカーをアクティブ状態にする
            this.focusMarker = marker;
            // マーカーにマウスオーバー時に地点名の情報ウィンドウを表示
            marker.addListener('mouseover', () => {
              if (this.$i18n.locale === 'ja') {
                infoWindowHover.setContent(
                  `<div class="mb-0"><h2>${this.spots[i].name}</h2></div>`
                );
              } else if (this.$i18n.locale === 'en') {
                infoWindowHover.setContent(
                  `<div class="mb-0"><h2>${this.spots[i].name_ens}</h2></div>`
                );
              }
              infoWindowHover.open(map, marker);
            });
            // マーカーからマウスアウトした時に地点名の情報ウィンドウを非表示
            marker.addListener('mouseout', () => {
              infoWindowHover.close(map, marker);
            });

            marker.addListener('click', () => {
              // アクティブ状態のマーカーがあれば行う処理
              if (this.focusMarker) {
                // アクティブ中のマーカーを通常iconに戻す
                this.focusMarker.setIcon({
                  url: standardIcon,
                });
                // アクティブ中のマーカーのバウンドを止める
                this.focusMarker.setAnimation(null);
              }
              // アクティブ中のマーカーのみ色を変更する
              marker.setIcon({
                url: focusIcon,
              });
              // アクティブ中のマーカーのみバウンドさせる
              marker.setAnimation(google.maps.Animation.BOUNCE);
              // 情報ウィンドウを固定で表示する
              if (this.$i18n.locale === 'ja') {
                infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name}</h2></div>`);
              } else if (this.$i18n.locale === 'en') {
                infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name_ens}</h2></div>`);
              }
              infoWindow.open(map, marker);
              // 地点に関する動画一覧ページに遷移する処理
              this.getVideo(marker, this.spot, this.area);
              // マーカー(地点)がクリックされた時にクリック数が+1カウントされる
              this.clickCount(this.spot, this.area);
            });
          } else {
            // 地点の緯度経度
            let spotLatLng = { lat: Number(this.spots[i].lat), lng: Number(this.spots[i].lng) };
            // 地点にマーカーをたてる
            marker_2[i] = new google.maps.Marker({
              position: spotLatLng,
              map,
              icon: standardIcon,
              animation: google.maps.Animation.DROP,
            });
            markers.push(marker_2[i]);
            // マーカークリック時に動画の情報ウィンドウを表示
            marker_2[i].addListener('click', () => {
              // アクティブ状態のマーカーがあれば行う処理
              if (this.focusMarker) {
                // アクティブ中のマーカーを通常iconに戻す
                this.focusMarker.setIcon({
                  url: standardIcon,
                });
                // アクティブ中のマーカーのバウンドを止める
                this.focusMarker.setAnimation(null);
              }
              // アクティブ中のマーカーのみ色を変更する
              marker_2[i].setIcon({
                url: focusIcon,
              });
              // アクティブ中のマーカーのみバウンドさせる
              marker_2[i].setAnimation(google.maps.Animation.BOUNCE);
              // 情報ウィンドウを固定で表示する
              if (this.$i18n.locale === 'ja') {
                infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name}</h2></div>`);
              } else if (this.$i18n.locale === 'en') {
                infoWindow.setContent(`<div class="mb-0"><h2>${this.spots[i].name_ens}</h2></div>`);
              }
              infoWindow.open(map, marker_2[i]);
              // 地点に関する動画一覧ページに遷移する処理
              this.getVideo(marker_2[i], this.spots[i], this.area);
              // マーカー(地点)がクリックされた時にクリック数が+1カウントされる
              this.clickCount(this.spots[i], this.area);
            });

            // マーカーにマウスオーバー時に地点名の情報ウィンドウを表示
            marker_2[i].addListener('mouseover', () => {
              if (this.$i18n.locale === 'ja') {
                infoWindowHover.setContent(
                  `<div class="mb-0"><h2>${this.spots[i].name}</h2></div>`
                );
              } else if (this.$i18n.locale === 'en') {
                infoWindowHover.setContent(
                  `<div class="mb-0"><h2>${this.spots[i].name_ens}</h2></div>`
                );
              }
              infoWindowHover.open(map, marker_2[i]);
            });
            // マーカーからマウスアウトした時に地点名の情報ウィンドウを非表示
            marker_2[i].addListener('mouseout', () => {
              infoWindowHover.close(map, marker_2[i]);
            });
          }
        }

        // マーカーを管理する Marker Clusterer を追加する
        const markerCluster = new MarkerClusterer({ markers, map });

        markerCluster.addListener('click', () => {
          // アクティブ状態のマーカーがあれば行う処理
          if (this.focusMarker) {
            // アクティブ中のマーカーを通常iconに戻す
            this.focusMarker.setIcon({
              url: standardIcon,
            });
            // アクティブ中のマーカーのバウンドを止める
            this.focusMarker.setAnimation(null);
          }
        });
      }
    }, 1000);
  },
  methods: {
    // 地点に関する動画一覧ページに遷移する処理
    getVideo(marker, spot, area) {
      // マーカーをアクティブ状態にする
      this.focusMarker = marker;
      // マーカーの情報ウィンドウに印をつける
      // マーカー(地点)がクリックされた時にクリック数が+1カウントされる
      this.$emit('get-video', spot, area);
    },
    clickCount(spot) {
      this.$emit('click-count', spot);
    },
  },
};
</script>

<style scoped>
.map {
  margin: auto;
  height: 400px;
  max-width: 600px;
}
</style>
