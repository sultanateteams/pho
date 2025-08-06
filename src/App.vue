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

        <!-- Variant 2: Web-based QR scanner -->
        <button @click="openScanner(index)">📷 Browser Scanner</button>
      </div>

      <div class="name-section">
        <label>Name {{ index + 1 }}:</label>
        <input type="text" v-model="complect.name" placeholder="Enter name" />
      </div>
    </div>

    <button class="add-btn" @click="addComplect">+ Add More</button>

    <div v-if="scanning" id="reader" class="camera-preview"></div>
  </div>
</template>

<script>
import jsQR from "jsqr";

export default {
  data() {
    return {
      complects: [{ barcode: "", name: "" }],
      scanning: false,
      scanner: null,
      currentScanIndex: null,
    };
  },
  methods: {
    // 📸 Variant 1: Use native camera
    async onImageSelected(event, index) {
      const file = event.target.files[0];
      if (!file) return;

      const img = new Image();
      img.src = URL.createObjectURL(file);

      img.onload = () => {
        const canvas = document.createElement("canvas");
        canvas.width = img.width;
        canvas.height = img.height;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const code = jsQR(imageData.data, imageData.width, imageData.height);

        if (code) {
          this.complects[index].barcode = code.data;
        } else {
          alert("QR kod topilmadi. Iltimos, boshqa rasmni urinib ko‘ring.");
        }
      };
    },

    // 📷 Variant 2: Web scanner (html5-qrcode)
    async openScanner(index) {
      this.currentScanIndex = index;
      this.scanning = true;

      this.$nextTick(async () => {
        if (!this.scanner) {
          const { Html5Qrcode } = await import("html5-qrcode");
          this.scanner = new Html5Qrcode("reader");
        }

        this.startScanner();
      });
    },

    startScanner() {
      this.scanner
        .start(
          { facingMode: "environment" },
          {
            fps: 15, // yuqoriroq FPS
            qrbox: { width: 300, height: 300 }, // barcode'lar uchun katta maydon
            aspectRatio: 1.7778, // 16:9
            disableFlip: true, // orqa kamera ustuvor
            experimentalFeatures: {
              useBarCodeDetectorIfSupported: true, // autofokus uchun Web API
            },
          },
          (decodedText) => {
            if (this.currentScanIndex !== null) {
              this.complects[this.currentScanIndex].barcode = decodedText;
              this.stopScanner();
            }
          },
          (errorMessage) => {
            // Har bir frame'da error bo'lishi mumkin, shunchaki e'tibor bermaymiz
          }
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
.camera-input {
  display: block;
  margin-top: 5px;
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
</style>
