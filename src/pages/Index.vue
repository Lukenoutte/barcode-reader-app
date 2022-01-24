<template>
  <div class="row items-center" style="height: 100vh">
    <div class="col text-center">
      <q-btn color="primary" icon="camera_alt" label="Ler código de barras"
        class="full-width" size="lg" @click="startScan()"
        v-show="!shouldShowScanner"/>
      <div class="text-h6" v-if="code">Codigo: {{ code }}</div>
      <div ref="scannerContainer" class="video-container"
      v-show="shouldShowScanner">
      </div>
      <q-page-sticky position="bottom-right" :offset="[18, 18]">
        <q-btn  icon="cancel" color="negative" label="Fechar" v-show="shouldShowScanner"
        @click="onStop" />
      </q-page-sticky>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import Quagga from '@ericblade/quagga2';
export default defineComponent({
  name: 'PageIndex',
  setup: () => {
    const code = ref('')
    const shouldShowScanner = ref(false)
    const scannerContainer = ref(null)
    const showVideo = ref(false)

    const startScan = () => {
      shouldShowScanner.value = true
      if (scannerContainer.value.children[0]) showVideo.value = true
      Quagga.init({
        inputStream : {
          name : "Live",
          type : "LiveStream",
          target: scannerContainer.value
        },
        decoder : {
          readers : ["ean_reader"]
        }
      }, function(err) {
          if (err) {
              console.log(err);
              return
          }
          console.log("Initialization finished. Ready to start");
          Quagga.start();
          Quagga.onDetected(onDetected)
      });
    }


    const onDetected = (data) => {
      code.value = data.codeResult.code
      shouldShowScanner.value = false
      onStop()
    }

    const onStop = () => {
      Quagga.stop()
      shouldShowScanner.value = false
    }

    return { startScan, shouldShowScanner, code, onStop, onDetected, scannerContainer, showVideo }
  }
})
</script>

<style scoped>

  .video-container {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 75%;
    overflow: hidden;
    object-fit: fill;
  }

  .video-container video {
    min-width: 100%;
    min-height: 75%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
</style>
