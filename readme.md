# js-QR-reader-sample

## 目的
QRコードリーダーをjavascriptで実装する簡単なサンプルが欲しかった

## 機能
* スマホのカメラやWebカメラでQRコードを値を取得する
* QRコードを認識したら、赤枠を表示する。

## 利用モジュール
* [jsQR](https://github.com/cozmo/jsQR)

## 使い方
下記ファイルを同一フォルダに配置して、ブラウザから開きます。
1. index.html
1. jsQR.js

※スマホの場合, httpsで開く必要があります

github pagesから試せます。
[index.html](https://murasuke.github.io/js-QR-reader/public/index.html)

### ローカルで試す場合
オレオレ証明書を作成してから、httpsを有効にしたhttp-serverを起動し、ブラウザから起動してください。
```
http-server -S -C cert.pem
Generating a RSA private key
.................................+++++
.......................+++++
～～略～～

npx http-server -S -C cert.pem
Starting up http-server, serving ./public through https
Available on:
  https://172.23.0.1:8080
  https://127.0.0.1:8080
```

## Licence
This software includes the work that is distributed in the Apache License 2.0