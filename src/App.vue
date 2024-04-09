<template>
  <input type="file" ref="fileInput" @change="handleFileUpload">
  <button @click="compareFiles">上傳檔案</button>
  <div class="file-info" v-if="file">已選擇檔案: {{ file.name }}</div>
  <div v-if="uploading">上傳中...</div>
  <div v-if="uploadSuccess">上傳成功!</div>
  <div v-if="uploadError">上傳失敗，請重試。</div>
  <div class="accuracy-info" v-if="showAcc">Accurancy : {{ accurancy }}%</div>
</template>

<script setup>
import { ref } from 'vue';

const file = ref(null);
const uploading = ref(false);
const uploadSuccess = ref(false);
const uploadError = ref(false);
const accurancy =  ref(null);
const showAcc = ref(false);
const handleFileUpload = (event) => {
const uploadedFile = event.target.files[0];
if (uploadedFile && uploadedFile.name.endsWith('.csv')) {
  file.value = uploadedFile;
} else {
  file.value = null;
  alert('請上傳 CSV 檔案');
}
};

const compareFiles = async () => {


try {
  const uploadedFileContent = await readFile(file.value);
  const answer = await readLocalFile();
  var correct = 0;
  var error = 0;
  for (let i = 0; i < answer.length; i++) {
    if(answer[i][1] == uploadedFileContent[i][1]) 
      correct = correct + 1
    else
      error = error + 1
  } 
  accurancy.value = (correct / answer.length)*100
  showAcc.value = true
  
  console.log(accurancy)
} catch (error) {
  console.error('讀取文件時發生錯誤：', error);
  uploadError.value = true;
  uploading.value = false;
}
};

const readFile = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.onload = (event) => {
      const content = event.target.result;
      const lines = content.split('\n');
      const rows = lines.map(line => line.split(','));
      resolve(rows);
    };

    reader.onerror = (error) => {
      reject(error);
    };

    reader.readAsText(file);
  });
};
const readLocalFile = async () => {
  const response = await fetch('src/assets/answer-forTA.csv');
  if (!response.ok) {
    throw new Error('無法讀取正確答案');
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder('utf-8');
  let result = [];
  let partial = '';
  let done = false;

  while (!done) {
    const { value, done: readerDone } = await reader.read();

    if (readerDone) {
      done = true;
      if (partial) {
        result.push(partial);
      }
      continue;
    }

    partial += decoder.decode(value, { stream: true });
    const lines = partial.split('\n');
    partial = lines.pop();

    for (const line of lines) {
      result.push(line.split(','));
    }
  }

  return result;
};

</script>

<style>
  #app {
    display: flex;
    flex-direction: column;
  }

  .file-info {
    margin-bottom: 10px;
  }

  .accuracy-info {
    margin-top: auto;
  }
</style>