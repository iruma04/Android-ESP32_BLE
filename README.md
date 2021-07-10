# Android - ESP32間でのBLE通信
### 開発環境
Android Studio v4.2.1  
* 言語: java
* ライブラリ: MPAndroidChart:v3.1.0  

Arduino IDE  
* 追加ボードマネージャーのURL: https://dl.espressif.com/dl/package_esp32_index.json  
* ボードマネージャー: ESP32  

### 概要
central = ESP32（マイコン）  
peripheral = Android  

セントラルとなるマイコンで加速度センサの数値を取得して、ペリフェラルとなるAndroidでデータを取得するプログラムとなっています。  

以降で、小分けにし詳しく説明します。  

# Android
## 1.BLEデバイスを検出&接続
※ BLEScanCallbackは実装していません  

ボタンのクリックメソッドで、ダイアログを表示しその結果を評価して検出&接続処理を行っています  

## 2.データの受け取り
characteristicのnotifyでデータを取得しています  

"x値, y値, z値"という形でデータを受け取り、","（コンマ）で分けてStringの配列に格納します  
そのデータを一つずつ取り出し、lineChartに表示させています  

たまに、配列のエラーが起きますが原因は調査中です  

## 3.線グラフに表示
MainActivityでchartを初期化し、横軸の最大値を決めないことである程度データがたまったら左側に古いデータが流れていく仕様になっています  

表示の仕方などについてはプログラムに書いてある通りです  

# ESP32
## 1.BLEの初期化について
UUIDを作成し定数として宣言

void setup()の「//BLEセットアップ」としてコメントアウトしてある部分より下側が初期化処理です
* デバイスの名前を設定
* サーバーを作成し、BLE接続を判断するためのcallbackを設定
* サービスのUUIDを設定し、characteristicのUUIDを設定し機能の初期化をする
* サービスをスタート
* 自分のデバイスがperipheralのScanに引っかかるようにAdvertisingの設定をし開始させる

## 2.loop内の処理
1つ目のif文
* Androidとの接続が完了したら割り込み処理の設定をする。  
* ボタンが押されたらboolの変数がtrueになるのでif文が通り、
データを取得する関数を呼び出し戻り値でデータを取得し間に","（コンマ）を仕込みながら一つの変数にまとめます。  
* String型の変数を".c_str()"でchar型に変更しnotifyでAndroidに通知する。

2つ目のif文  
* 接続が終了したときに通り、LEDを変化させ新しくAdvertisingを開始  

3つ目のif文  
* 接続が開始された時に通り、booleanの変数を変化させることで「2つ目のif文」を補助しています

## 3.その他機能
RGBLEDを使用し、BLEの接続状況などを視覚的にわかるようにしています  

3軸加速度センサを使用しています  

GPIO2にボタンを接続し、割り込み処理を実行しています
