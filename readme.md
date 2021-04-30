# js-QR-reader-sample

## 目的
html＋javascriptだけで実装されたシンプルなQRコードリーダーのサンプル(が欲しかった)

## 機能
1. スマホのカメラやWebカメラでQRコードを値を取得する。
1. QRコードを認識したら、赤枠を表示する。

## 利用モジュール
* [jsQR](https://github.com/cozmo/jsQR)

## 使い方
下記ファイルを同一フォルダに配置して、ブラウザから開きます(要Webカメラ)。
1. index.html
1. jsQR.js

* https://github.com/murasuke/js-QR-reader

※スマホの場合, httpsで開く必要があります

* github pagesから試せます。

  [index.html](https://murasuke.github.io/js-QR-reader/public/index.html)

### ローカルで試す場合
httpsが必要なため、オレオレ証明書を作成してから、http-serverを起動してください。
```
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
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
----
## ソース解説
### `<head>`部分
* QR読み取りライブラリをロードする
```html
<script src="./jsQR.js"></script>
```
### `<body>`部分
* '<div id="result" ～>'
  * QR読み取り結果を表示
* `<video>～</video>`
  * カメラ映像出力部分。z-indexにマイナスを指定して、オーバーレイの奥に表示します。
* `<div id="overlay" >～</div>`
  * QRコードを赤枠で囲うためのオーバーレイ。position:relative;で重ねています。
```html
  <div id="result" style="min-height: 20px;"></div>
  <div>
    <div style="position:relative;">
      <video style="position: absolute; z-index: -100;"></video>
      <div id="overlay" style="position: absolute; border: 1px solid #F00;"></div>
    </div>      
  </div>
```
### `<script>`部分
* jsQRで認識したQRコードの箇所を赤枠で囲むための関数
  * id="overlay"をQRコードの位置へ移動します。
```javascript
  const drawRect = (topLeft, bottomRight) => {
    const { x: x1, y: y1 } = topLeft;
    const { x: x2, y: y2 }= bottomRight;

    const overlay = document.querySelector('#overlay');
    overlay.style.left = `${x1}px`;
    overlay.style.top =`${y1}px`;
    overlay.style.width = `${x2 - x1}px`;
    overlay.style.height =`${y2 - y1}px`;
  };
```

* カメラの準備(videoタグに表示)
```javascript
  const stream = await navigator.mediaDevices.getUserMedia(constraints);
  const video = document.querySelector('video');
  video.srcObject = stream;
  video.play();
```


* canvas作成
  * 描画負荷を軽減するためOffscreenCanvasを作成します。
  * QRコード認識で必要なイメージを、videoから作成するために利用します。
```javascript
  const { width, height } = constraints.video;
  const canvas = new OffscreenCanvas(width, height);
  const context = canvas.getContext('2d');
```


* QRコード認識
  * videoからイメージデータを作成し、QR認識処理に引き渡します。
  * 認識した場合は、コードの表示と赤枠の表示を行います。
```javascript
  const timer = setInterval(() => {
      context.drawImage(video, 0, 0, width, height);
      const imageData = context.getImageData(0, 0, width, height);
      const code = jsQR(imageData.data, imageData.width, imageData.height);
      if (code) {
        document.querySelector('#result').textContent = code.data;
        drawRect(code.location.topLeftCorner, code.location.bottomRightCorner);                
      } else {
        document.querySelector('#result').textContent = '';
      }
    }, 300);
```


* 全体
```html
<!DOCTYPE html>

<html lang="ja">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>JS QR Code Reader</title>
    <meta name="description" content="QR Code Reader" />
    <script src="./jsQR.js"></script>
  </head>
  <body>
    <div id="result" style="min-height: 20px;"></div>
    <div>
      <div style="position:relative;">
        <video style="position: absolute; z-index: -100;"></video>
        <div id="overlay" style="position: absolute; border: 1px solid #F00;"></div>
      </div>      
    </div>

    <script>
      const constraints = { 
        audio: false, 
        video: {
          facingMode: 'environment', 
          width: 500, 
          height: 500, 
      }};

      const drawRect = (topLeft, bottomRight) => {
        const { x: x1, y: y1 } = topLeft;
        const { x: x2, y: y2 }= bottomRight;

        const overlay = document.querySelector('#overlay');
        overlay.style.left = `${x1}px`;
        overlay.style.top =`${y1}px`;
        overlay.style.width = `${x2 - x1}px`;
        overlay.style.height =`${y2 - y1}px`;
      };

      (async() => {
        try {
          const stream = await navigator.mediaDevices.getUserMedia(constraints);
          const video = document.querySelector('video');
          video.srcObject = stream;
          video.play();

          const { width, height } = constraints.video;
          const canvas = new OffscreenCanvas(width, height);
          const context = canvas.getContext('2d');
       
          const timer = setInterval(() => {
              context.drawImage(video, 0, 0, width, height);
              const imageData = context.getImageData(0, 0, width, height);
              const code = jsQR(imageData.data, imageData.width, imageData.height);
              if (code) {
                document.querySelector('#result').textContent = code.data;
                drawRect(code.location.topLeftCorner, code.location.bottomRightCorner);                
              } else {
                document.querySelector('#result').textContent = '';
              }
            }, 300);
        } catch(error) {
          console.log('load error', error);
        }
      })();
    </script>
  </body>
</html>
```

## Licence
This software includes the work that is distributed in the Apache License 2.0
