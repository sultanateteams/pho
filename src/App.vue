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
        <button @click="openScanner(index)">📷 Scan</button>
      </div>

      <div class="name-section">
        <label>Name {{ index + 1 }}:</label>
        <input type="text" v-model="complect.name" placeholder="Enter name" />
      </div>
    </div>

    <button class="add-btn" @click="addComplect">+ Add More</button>

    <!-- Only render scanner box when scanning -->
    <div v-if="scanning" id="reader" class="camera-preview"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      complects: [
        { barcode: "", name: "" }
      ],
      scanning: false,
      scanner: null,
      currentScanIndex: null,
    };
  },
  methods: {
    async openScanner(index) {
      this.currentScanIndex = index;
      this.scanning = true;

      this.$nextTick(async () => {
        // Ensure #reader element is in the DOM
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
          { fps: 10, qrbox: { width: 250, height: 250 } },
          (decodedText) => {
            if (this.currentScanIndex !== null) {
              this.complects[this.currentScanIndex].barcode = decodedText;
              this.stopScanner();
            }
          },
          (errorMessage) => {
            // Handle scan error (optional)
          }
        )
        .catch((err) => {
          console.error("Error starting scanner:", err);
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
      console.log("Saving complects:", this.complects);
    }
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
</style>
