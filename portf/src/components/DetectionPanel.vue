<template>
  <div class="panel">
    <h2>Deepfake Detection</h2>
    
    <div class="image-preview-box">
      <img v-if="uploadedImagePreview" :src="uploadedImagePreview" alt="업로드된 이미지"/>
      <div v-else class="placeholder">
        <span>탐지할 이미지를 첨부해주세요.</span>
      </div>
    </div>

    <div class="result-box">
      <strong>결과:</strong>
      <div v-if="isLoading" class="loading-indicator">
        <div class="spinner-small"></div>
        <span>분석 중입니다...</span>
      </div>
      <span v-else-if="deepfakeResult">{{ deepfakeResult }} &nbsp; &nbsp; &nbsp; <strong>확률:</strong> {{ probability }}</span>
      <span v-else>-</span>
    </div>

    <div class="actions">
      <input type="file" @change="handleFileUpload" accept="image/*" ref="fileInput" style="display: none;" />
      <button class="button-secondary" @click="triggerFileUpload">파일 첨부</button>
      <button @click="detectDeepfake" :disabled="!uploadedFile || isLoading">
        {{ isLoading ? '탐지 중...' : '탐지 실행' }}
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'DetectionPanel',
  data() {
    return {
      uploadedFile: null,
      uploadedImagePreview: null,
      deepfakeResult: null,
      probability: null,
      isLoading: false,
      // apiBaseUrl: 'http://localhost:2000',
      apiBaseUrl: 'https://79b1bea58ae5.ngrok-free.app',
    };
  },
  methods: {
    triggerFileUpload() {
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;
      this.uploadedFile = file;
      this.uploadedImagePreview = URL.createObjectURL(file);
      this.deepfakeResult = null;
      this.probability = null;
    },
    async detectDeepfake() {
      if (!this.uploadedFile) {
        alert("탐지할 이미지 파일을 먼저 첨부해주세요.");
        return;
      }
      
      this.$emit('update:error', null); // 부모의 에러 메시지 초기화
      this.deepfakeResult = null;
      this.probability = null;
      this.isLoading = true;

      const formData = new FormData();
      formData.append('userAccount', "TempAccount");
      formData.append('data', this.uploadedFile);

      try {
        const response = await axios.post(`${this.apiBaseUrl}/TestAPI`, formData, {
          headers: { 'Content-Type': 'multipart/form-data' }
        });
        
        this.deepfakeResult = response.data.userAccount;
        let probValue = Math.ceil(response.data.data * 10000) / 100;
        this.probability = probValue + "%";

      } catch (err) {
        console.error("Deepfake 탐지 API 호출 오류:", err);
        this.$emit('update:error', "딥페이크 탐지에 실패했습니다."); // 부모로 에러 전파
        this.deepfakeResult = '탐지 실패';
      } finally {
        this.isLoading = false;
      }
    }
  }
}
</script>

<style scoped>
/* DetectionPanel에만 적용되는 스타일 */
.result-box {
  text-align: center;
  min-height: 48px;
  display: flex;
  align-items: center;
  font-size: 16px;
}

.result-box strong {
  margin-right: 8px;
}

.loading-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--secondary-color);
  font-size: 15px;
}

.spinner-small {
  border: 3px solid #f0f2f5;
  border-top: 3px solid var(--secondary-color);
  border-radius: 50%;
  width: 16px;
  height: 16px;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>