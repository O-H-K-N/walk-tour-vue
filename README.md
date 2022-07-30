# VtourHub

![VtourHub](https://user-images.githubusercontent.com/81758321/181432581-694bcbc5-3d4c-47b0-bd7e-a79180acf9b5.png)

## □ サービスURL
https://vtourhub.jp/

## □ リポジトリ

### フロントエンド
https://github.com/O-H-K-N/walk-tour-vue

### バックエンド
https://github.com/O-H-K-N/walk-tour-rails

## □ サービス概要
いつでもどこでも旅行気分を味わうことができるサービスです。

## □ ユーザーが抱える課題
- コロナ禍で国内含め旅行がしにくい
- 旅行ができても、時間や安全性の都合で街の隅々を歩き回ることができない
- 行ってみたい国があり、あらかじめ現地の雰囲気を体感したい時に、情報がない時がある

## □ サービスを開発した背景
コロナ禍になり国内外の渡航が難しくなったため、いつでもどこでも旅行気分を味わうことができるサービスをつくりたいと考え開発しました。Youtube上にある#4kwalk, #walkingtourなどのバーチャルツアーの動画を活用し、まるで世界中の街中にいるかのように、雰囲気を体感できるサービスを作ることができるのではないかと考えました。

## □ 主な使用予定技術
### バックエンド
- Ruby 3.1.2
- Rails(APIモード) 6.1.6
- 主なGem
  - sorcery
  - jwt
  - rack-cors
  - google-api-client
  - rubocop
- 主なライブラリ
  - AdminLTE 3.0
  
### フロントエンド
- Vue.js 2.6.14
- Vuetify 2.6.0
- 主なライブラリ
  - vue-router
  - vuex
  - axios
  - jqvmap

### 外部API
- YouTube Data API
- Geocoding API
- Maps JavaScript API

### インフラ
- PostgreSQL
- Heroku
- Firebase Hosting

## □ ER図

![スクリーンショット 2022-07-28 15 34 48](https://user-images.githubusercontent.com/81758321/181437090-edbbb467-8be9-407a-bf01-b716bba9b7b0.png)


## □ 画面遷移図
[figmaで作成した画面遷移図](https://www.figma.com/file/JhXUkEsOpPW3FJeYJXeJCo/%E7%94%BB%E9%9D%A2%E9%81%B7%E7%A7%BB%E5%9B%B3?node-id=0%3A1)

## □ 今後の実装予定機能
- Twiiterログイン
- 多言語化

