<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Clean Video Call</title>

<style>
body {
  margin:0;
  background:#0f1115;
  color:white;
  font-family:Segoe UI, Arial;
  display:flex;
  flex-direction:column;
  align-items:center;
  height:100vh;
}

#status {
  margin:10px;
  color:#00e5ff;
}

#callArea {
  position:relative;
  width:90%;
  height:70%;
  background:#000;
  border-radius:16px;
  overflow:hidden;
  box-shadow:0 0 30px #000;
}

#remoteVideo {
  width:100%;
  height:100%;
  object-fit:cover;
}

#localVideo {
  position:absolute;
  bottom:15px;
  right:15px;
  width:180px;
  height:120px;
  object-fit:cover;
  border-radius:12px;
  border:2px solid #00e5ff;
  background:black;
}

#controls {
  margin-top:15px;
}

button {
  padding:12px 20px;
  margin:5px;
  font-size:15px;
  border:none;
  border-radius:50px;
  cursor:pointer;
}

.green { background:#00c853; }
.gray { background:#424242; color:white; }
.red { background:#d50000; color:white; }

button:hover {
  opacity:0.85;
}
</style>
</head>

<body>

<h2>📹 Video Call</h2>
<div id="status">Waiting for camera…</div>

<div id="callArea">
  <video id="remoteVideo" autoplay></video>
  <video id="localVideo" autoplay muted></video>
</div>

<div id="controls">
  <button class="green" onclick="createOffer()">Start Call</button>
  <button class="gray" onclick="toggleMic()">Mute</button>
  <button class="gray" onclick="toggleCam()">Camera</button>
  <button class="red" onclick="endCall()">End</button>
</div>

<script>
const statusText = document.getElementById("status");
const localVideo = document.getElementById("localVideo");
const remoteVideo = document.getElementById("remoteVideo");

let micOn = true;
let camOn = true;
let localStream;

const pc = new RTCPeerConnection({
  iceServers:[{urls:"stun:stun.l.google.com:19302"}]
});

pc.ontrack = e => {
  remoteVideo.srcObject = e.streams[0];
  statusText.textContent = "Connected";
};

navigator.mediaDevices.getUserMedia({video:true,audio:true})
.then(stream=>{
  localStream = stream;
  localVideo.srcObject = stream;
  stream.getTracks().forEach(t=>pc.addTrack(t,stream));
  statusText.textContent = "Ready to start call";
});

pc.onicecandidate = e=>{
  if(!e.candidate){
    console.log("SEND THIS TO FRIEND:");
    console.log(JSON.stringify(pc.localDescription));
  }
};

async function createOffer(){
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);
  statusText.textContent = "Call started — send code to friend";
}

function toggleMic(){
  micOn = !micOn;
  localStream.getAudioTracks()[0].enabled = micOn;
}

function toggleCam(){
  camOn = !camOn;
  localStream.getVideoTracks()[0].enabled = camOn;
}

function endCall(){
  pc.close();
  remoteVideo.srcObject = null;
  statusText.textContent = "Call ended";
}
</script>

</body>
</html>
