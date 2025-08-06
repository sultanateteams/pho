<template>
  <div class="scanner-app">
    <button @click="saveAll">Hammasini saqlash</button>
    <h2 class="title">Barcode Complects</h2>

    <div
      v-for="(complect, index) in complects"
      :key="index"
      class="complect-box"
    >
      <div class="barcode-section">
        <label>Barcode {{ index + 1 }}:</label>
        <input type="text" v-model="complect.barcode" readonly />
        <button @click="openScanner(index)">📷 Browser Scanner</button>
      </div>

      <div class="name-section">
        <label>Name {{ index + 1 }}:</label>
        <input type="text" v-model="complect.name" placeholder="Enter name" />
      </div>
    </div>

    <button class="add-btn" @click="addComplect">+ Add More</button>

    <div v-if="scanning" class="scanner-overlay">
      <button class="switch-btn" @click="toggleCamera">🔁 Kamera almashtirish</button>
      <div id="reader" class="camera-preview"></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      complects: [{ barcode: "", name: "" }],
      scanning: false,
      scanner: null,
      currentScanIndex: null,
      cameraFacing: "environment", // or "user"
    };
  },
  methods: {
    async openScanner(index) {
      this.currentScanIndex = index;
      this.scanning = true;

      this.$nextTick(async () => {
        const { Html5Qrcode } = await import("html5-qrcode");

        if (!this.scanner) {
          this.scanner = new Html5Qrcode("reader");
        }

        this.startScanner();
      });
    },

    startScanner() {
      this.scanner
        .start(
          { facingMode: this.cameraFacing },
          {
            fps: 15,
            qrbox: { width: 300, height: 300 },
            aspectRatio: 1.7778,
            disableFlip: true,
            experimentalFeatures: {
              useBarCodeDetectorIfSupported: true,
            },
          },
          (decodedText) => {
            if (this.currentScanIndex !== null) {
              this.complects[this.currentScanIndex].barcode = decodedText;
              this.stopScanner();
            }
          },
          () => {}
        )
        .catch((err) => {
          console.error("Scanner error:", err);
        });
    },

    stopScanner() {
      if (this.scanner) {
        this.scanner.stop().then(() => {
          this.scanning = false;
          this.currentScanIndex = null;
        });
      }
    },

    restartScanner() {
      if (this.scanner) {
        this.scanner.stop().then(() => {
          this.startScanner(); // facingMode yangilangan holda qayta ishga tushadi
        });
      }
    },

    toggleCamera() {
      this.cameraFacing =
        this.cameraFacing === "environment" ? "user" : "environment";
      this.restartScanner();
    },

    addComplect() {
      this.complects.push({ barcode: "", name: "" });
    },

    saveAll() {
      console.log("Saqlanayotgan complectlar:", this.complects);
    },
  },
};
</script>

<style scoped>
.scanner-app {
  max-width: 600px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}
.title {
  text-align: center;
  margin-bottom: 20px;
}
.complect-box {
  border: 1px solid #ccc;
  padding: 15px;
  margin-bottom: 20px;
  border-radius: 8px;
}
.barcode-section,
.name-section {
  margin-bottom: 10px;
}
label {
  display: block;
  margin-bottom: 5px;
}
input[type="text"] {
  width: calc(100% - 10px);
  padding: 8px;
  margin-right: 10px;
}
button {
  padding: 8px 12px;
  font-size: 14px;
  cursor: pointer;
}
.add-btn {
  display: block;
  margin: 0 auto;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
}
.camera-preview {
  margin-top: 20px;
}
.scanner-overlay {
  margin-top: 20px;
  position: relative;
  text-align: center;
}
.switch-btn {
  margin-bottom: 10px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 4px;
}
</style>
