<template>
  <div class="container">
    <h1 class="mt-4">QRIS Statis to Dinamis Converter</h1>
    <div class="form-group">
      <label for="qrisInput">Input Data QRIS:</label>
      <input type="text" class="form-control" id="qrisInput" v-model="qris">
    </div>
    <div class="form-group">
      <label for="nominalInput">Input Nominal:</label>
      <input type="text" class="form-control" id="nominalInput" v-model="qty">
    </div>
    <div class="form-group">
      <label for="layananCheckbox">Biaya Layanan?</label>
      <input type="checkbox" id="layananCheckbox" v-model="isLayananEnabled">
    </div>
    <div v-if="isLayananEnabled" class="form-group">
      <label for="layananOption">Rupiah atau Persen?</label>
      <select class="form-control" id="layananOption" v-model="layananOption">
        <option value="r">Rupiah</option>
        <option value="p">Persen</option>
      </select>
    </div>
    <div v-if="isLayananEnabled && layananOption === 'r'" class="form-group">
      <label for="biayaLayananInput">Input Biaya Layanan Dalam Rupiah:</label>
      <input type="text" class="form-control" id="biayaLayananInput" v-model="tax">
    </div>
    <div v-if="isLayananEnabled && layananOption === 'p'" class="form-group">
      <label for="biayaLayananInput">Input Biaya Layanan Dalam Persen:</label>
      <input type="text" class="form-control" id="biayaLayananInput" v-model="tax">
    </div>
    <button class="btn btn-primary" @click="convertQRIS">Convert</button>
    <div v-if="result" class="mt-4">
      <p><strong>Result:</strong></p>
      <img :src="generateQRCode(result)" alt="QRIS Image">
    </div>
  </div>
</template>

<script>
import QRCode from 'qrcode';

export default {
  data() {
    return {
      qris: '',
      qty: '',
      isLayananEnabled: false,
      layananOption: 'r',
      tax: '',
      result: ''
    };
  },
  methods: {
    async convertQRIS() {
      const qris = this.qris;
      const qty = this.qty;
      const yn = this.isLayananEnabled ? 'y' : 'n';
      const tax = this.tax;

      // Implementasi logika konversi QRIS
      const convertQris = (qris, qty, tax = '') => {
        qris = qris.slice(0, -4);
        const step1 = qris.replace("010211", "010212");
        const step2 = step1.split("5802ID");
        let uang = "54" + qty.length.toString().padStart(2, '0') + qty;

        if (tax === '') {
          uang += "5802ID";
        } else {
          uang += tax + "5802ID";
        }

        const fix = step2[0] + uang + step2[1];
        const result = fix + convertCRC16(fix);

        return result;
      };

      // Implementasi fungsi CRC16
      const convertCRC16 = (str) => {
        let crc = 0xFFFF;
        const strlen = str.length;

        for (let c = 0; c < strlen; c++) {
          crc ^= str.charCodeAt(c) << 8;
          for (let i = 0; i < 8; i++) {
            if (crc & 0x8000) {
              crc = (crc << 1) ^ 0x1021;
            } else {
              crc = crc << 1;
            }
          }
        }

        let hex = crc & 0xFFFF;
        hex = hex.toString(16).toUpperCase();
        if (hex.length === 3) {
          hex = "0" + hex;
        }

        return hex;
      };

      // Memanggil fungsi konversi QRIS dengan parameter yang sesuai
      const result = convertQris(qris, qty, yn === 'y' ? tax : '');

      // Mengupdate hasil pada state 'result'
      this.result = result;
    },

    generateQRCode(data) {
      const canvas = document.createElement('canvas');
      QRCode.toCanvas(canvas, data, { errorCorrectionLevel: 'H' });
      return canvas.toDataURL();
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}
</style>
