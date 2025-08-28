<template>
  <div class="panel">
    <h2>Create Image</h2>
    
    <div class="image-preview-box">
      <div v-if="isLoading" class="loader-container">
        <div class="loader"></div>
        <span>이미지를 생성 중입니다...</span>
      </div>
      <!-- <img v-else-if="generatedImageUrl" :src="generatedImageUrl" alt="생성된 이미지"/> -->
      <img v-else-if="generatedImageUrl" :src="generatedImageUrl" alt="생성된 이미지"/>
      <div v-else class="placeholder">
        <span>생성된 이미지가 여기에 표시됩니다.</span>
      </div>
    </div>
    
    <div class="command-input">
      <input 
        type="text" 
        v-model="imagePrompt" 
        placeholder="생성할 이미지에 대한 커맨드(프롬프트) 입력 ---> 영어로,  high quality 를 포함해주세요"
        @keyup.enter="generateImage"
        :disabled="isLoading"
      />
    </div>

    <div class="actions">
      <button @click="generateImage" :disabled="!imagePrompt || isLoading">
        {{ isLoading ? '생성 중...' : '생성' }}
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'CreationPanel',
  data() {
    return {
      imagePrompt: '',
      Account:'',
      generatedImageUrl: '',
      isLoading: false,
      // apiBaseUrl: 'http://localhost:2000',
      apiBaseUrl: 'https://79b1bea58ae5.ngrok-free.app',
    };
  },
  methods: {
    async generateImage() {
      if (!this.imagePrompt.trim()) {
        alert("이미지 생성 커맨드를 입력해주세요.");
        return;
      }

      this.$emit('update:error', null); // 부모의 에러 메시지 초기화
      this.isLoading = true;
      this.Account = null;
      this.generatedImageUrl = '';

      const payload = {
        Account: "TempPrompt",
        Prompt: this.imagePrompt,
      };

      try {
        const response = await axios.post(`${this.apiBaseUrl}/PromptAPI`, payload);
        console.log(response.data.generatedImageUrl)
        this.generatedImageUrl = 'data:image/jpeg;base64,'+response.data.generatedImageUrl;
      } catch (err) {
        console.error("이미지 생성 API 호출 오류:", err);
        this.$emit('update:error', "이미지 생성에 실패했습니다."); // 부모로 에러 전파
      } finally {
        this.isLoading = false;
      }
    }
  }
}
</script>

<!-- <script setup>
import axios from 'axios';
import {ref} from 'vue';
defineOptions({ name: 'CreationPanel'});
const imagePrompt =ref('');
const Account = ref('');
const generatedImageUrl=ref(null);
const isLoading = ref(false);
const apiBaseUrl= "http://localhost:2000" /// 여기 BackEnd 주소

const emit = defineEmits(['update:error']);
const ganerateImage = async() =>{
  if(!imagePrompt.value.trim()){
    alert("이미지 생성 커맨드를 입력해주세요!!")
    return;
  }

emit('update:error',null) // 부모의 에러 메시지 초기화

isLoading.value=true;
Account.value=null;
generatedImageUrl.value=null;

const payload = {
  Account: Account.value,
  Prompt: imagePrompt.value,
}

try{
  const response = await axios.post(`${apiBaseUrl}/PromptAPI`,payload);
  console.log(response.data.generatedImageUrl);
}catch (err){
  console.error("이미지 생성 API 호출 오류:", err);
  emit('update:error',"이미지 생성에 실패했습니다.");
}finally{
  isLoading.value=false;
}
}
</script> -->


<style scoped>
/* CreationPanel에만 적용되는 스타일 */
.loader-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #888;
  font-size: 16px;
}

.loader {
  border: 6px solid #f0f2f5;
  border-top: 6px solid var(--primary-color);
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.command-input {
  width: 100%;
  box-sizing: border-box;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  margin-bottom: 16px;
  background-color: #fafafa;
}

.command-input input {
  width: 100%;
  border: none;
  outline: none;
  background: transparent;
  font-size: 16px;
}
</style>