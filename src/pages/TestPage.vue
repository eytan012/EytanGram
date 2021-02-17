<template>
  <div class="test">
  <video id="video" controls autoplay playsInline></video>
  <div>
    <button id="btn-front">Selfie</button>
    <button id="btn-back">Main</button>
  </div>
  </div>
</template>

<script>
window.onload=function() {


  (() => {
    const videoElm = document.querySelector('#video');
    const btnFront = document.querySelector('#btn-front');
    const btnBack = document.querySelector('#btn-back');

    const supports = navigator.mediaDevices.getSupportedConstraints();
    if (!supports['facingMode']) {
      alert('Browser Not supported!');
      return;
    }

    let stream;

    const capture = async facingMode => {
      const options = {
        audio: false,
        video: {
          facingMode,
        },
      };

      try {
        if (stream) {
          const tracks = stream.getTracks();
          tracks.forEach(track => track.stop());
        }
        stream = await navigator.mediaDevices.getUserMedia(options);
      } catch (e) {
        alert(e);
        return;
      }
      videoElm.srcObject = null;
      videoElm.srcObject = stream;
      videoElm.play();
    }

    btnBack.addEventListener('click', () => {
      capture('environment');
    });

    btnFront.addEventListener('click', () => {
      capture('user');
    });
  })();
}
export default {
  name: "TestPage"
}
</script>

<style scoped>
* {
  padding: 0;
  margin: 0;
}

video {
  max-width: 100vw;
}

button {
  width: 60px;
  margin-right: 20px;
}
</style>
