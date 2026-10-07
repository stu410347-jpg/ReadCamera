<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QR Code 掃描測試</title>

<style>
body{
  font-family: Arial, sans-serif;
  text-align:center;
  background:#f5f5f5;
  margin:0;
  padding:20px;
}

h2{
  margin-bottom:20px;
}

video{
  width:90%;
  max-width:500px;
  border-radius:10px;
  background:#000;
}

button{
  padding:10px 20px;
  margin:10px;
  font-size:16px;
  cursor:pointer;
}

#resultBox{
  margin:20px auto;
  width:90%;
  max-width:500px;
  background:#fff;
  border-radius:10px;
  padding:15px;
  box-shadow:0 2px 8px rgba(0,0,0,0.1);
  text-align:left;
  word-break:break-all;
}

#result{
  color:#d63384;
  font-weight:bold;
  font-size:18px;
}

#status{
  color:#666;
  margin-top:10px;
}

canvas{
  display:none;
}
</style>

<!-- jsQR 函式庫 -->
<script src="https://cdn.jsdelivr.net/npm/jsqr/dist/jsQR.js"></script>
</head>

<body>

<h2>QR Code 掃描測試</h2>

<video id="video" autoplay playsinline></video>
<canvas id="canvas"></canvas>

<br>

<button onclick="startCamera()">開啟攝影機</button>
<button onclick="stopCamera()">關閉攝影機</button>

<div id="resultBox">
  <div><strong>掃描結果：</strong></div>
  <div id="result">尚未讀到 QR Code</div>
  <div id="status">請先開啟攝影機</div>
</div>

<script>
let stream = null;
let scanInterval = null;
let lastResult = "";

const video = document.getElementById("video");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
const result = document.getElementById("result");
const statusText = document.getElementById("status");

async function startCamera(){
  try{
    stopCamera();

    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: "environment"
      },
      audio: false
    });

    video.srcObject = stream;
    statusText.textContent = "攝影機已開啟，請將 QR Code 對準鏡頭";

    video.onloadedmetadata = () => {
      video.play();
      startScan();
    };

  }catch(err){
    console.error(err);
    alert("無法開啟攝影機");
    statusText.textContent = "無法開啟攝影機";
  }
}

function startScan(){
  scanInterval = setInterval(() => {
    if(video.readyState === video.HAVE_ENOUGH_DATA){
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height);

      if(code){
        if(code.data !== lastResult){
          lastResult = code.data;
          result.textContent = code.data;
          statusText.textContent = "已成功讀到 QR Code";
          console.log("QR Code 內容：", code.data);
        }
      }
    }
  }, 300);
}

function stopCamera(){
  if(scanInterval){
    clearInterval(scanInterval);
    scanInterval = null;
  }

  if(stream){
    stream.getTracks().forEach(track => track.stop());
    stream = null;
  }

  video.srcObject = null;
  statusText.textContent = "攝影機已關閉";
}

window.addEventListener("beforeunload", () => {
  stopCamera();
});
</script>

</body>
</html>
