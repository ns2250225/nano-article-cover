<script setup lang="ts">
import { ref } from 'vue';

const title = ref('');
const loading = ref(false);
const progress = ref(0);
const resultImage = ref('');
const errorMsg = ref('');
const showPreview = ref(false);

const generateImage = async () => {
  if (!title.value) return;
  
  loading.value = true;
  progress.value = 0;
  resultImage.value = '';
  errorMsg.value = '';

  const promptTemplate = `[${title.value}] 主题海报  
 生成一张与主题相关的海报，写上主题相关的文字和插画，具体要求如下  
 - 文字：  
   - 重点文字用背景色或波浪线突出  
   - 普通文字标准黑色墨迹  
   - 强调文字用不同颜色标注  
 - 插画风格：  
   - 手绘简笔插图或杂志剪贴风  
   - 插图上可有钢笔风格涂鸦批注  
 - 批注元素：  
   - 使用马克笔效果，模拟手写批注和随意涂鸦  
   - 可加入拼贴风格照片并进行涂鸦或备注  
 - 文本要求：  
   - 中文标准规范，语法准确  
   - 确保用字规范，避免错字或异体字  
 - 海报要求:  
   - 比例2.35:1`;

  try {
    const response = await fetch('https://grsai.dakka.com.cn/v1/draw/nano-banana', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer sk-b7182e2c0c3248b6aafcedad465af768',
        'Accept': 'text/event-stream'
      },
      body: JSON.stringify({
        model: "nano-banana-pro",
        prompt: promptTemplate,
        aspectRatio: "16:9",
        imageSize: "1K",
        urls: []
      })
    });

    if (!response.ok) {
       throw new Error(`HTTP error! status: ${response.status}`);
    }

    if (!response.body) {
      throw new Error('No response body');
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = '';

    while (true) {
      const { done, value } = await reader.read();
      
      if (value) {
        buffer += decoder.decode(value, { stream: !done });
      }
      
      const lines = buffer.split('\n');
      
      if (!done) {
          buffer = lines.pop() || '';
      } else {
          buffer = '';
      }

      for (const line of lines) {
        const trimmed = line.trim();
        if (!trimmed) continue;
        
        // console.log('Received line:', trimmed); // Debug log

        try {
          let jsonStr = trimmed;
          // Handle SSE format (data: ...)
          if (jsonStr.startsWith('data:')) {
             jsonStr = jsonStr.replace(/^data:\s*/, '');
          }
          
          if (jsonStr === '[DONE]') continue;
          if (!jsonStr) continue;

          const data = JSON.parse(jsonStr);
          // console.log('Parsed data:', data); // Debug log
          
          if (typeof data.progress !== 'undefined') {
            progress.value = data.progress;
          }
          
          if (data.results && data.results.length > 0 && data.results[0].url) {
            resultImage.value = data.results[0].url;
          }
          
          if (data.status === 'succeeded') {
            loading.value = false;
            progress.value = 100;
            return;
          } else if (data.status === 'failed') {
            const reason = data.failure_reason || data.error || '生成失败';
            try { await reader.cancel(); } catch (e) { /* ignore cancel error */ }
            throw new Error(`${reason}，建议您稍后重试`);
          }
        } catch (e) {
          // console.error('Parse error for line:', trimmed, e);
          // Ignore parsing errors for partial lines
          if (e instanceof Error && e.message.includes('建议您稍后重试')) {
            throw e;
          }
        }
      }
      
      if (done) break;
    }
  } catch (err: any) {
    console.error(err);
    errorMsg.value = err.message || '生成失败，请稍后重试';
    loading.value = false;
    progress.value = 0; // Reset progress on error
  }
};

const downloadImage = async (url: string) => {
  try {
    const response = await fetch(url);
    const blob = await response.blob();
    const blobUrl = window.URL.createObjectURL(blob);
    
    const link = document.createElement('a');
    link.href = blobUrl;
    link.download = `cover-${Date.now()}.png`;
    document.body.appendChild(link);
    link.click();
    
    document.body.removeChild(link);
    window.URL.revokeObjectURL(blobUrl);
  } catch (e) {
    console.error('Download failed:', e);
    // Fallback to opening in new tab if fetch fails (e.g. CORS)
    window.open(url, '_blank');
  }
};
</script>

<template>
  <div class="min-h-screen bg-gray-50 py-6 sm:py-12 px-4 sm:px-6 lg:px-8 font-sans">
    <div class="max-w-3xl mx-auto">
      <div class="text-center mb-8 sm:mb-12">
        <h1 class="text-3xl font-extrabold text-indigo-600 tracking-tight sm:text-4xl md:text-5xl">
          Nano Article Cover
        </h1>
        <p class="mt-3 text-lg sm:text-xl text-gray-500">
          智能生成文章封面，让您的文章脱颖而出
        </p>
      </div>

      <div class="bg-white shadow-xl rounded-xl sm:rounded-xl p-4 sm:p-8 transition-all hover:shadow-2xl">
        <div class="mb-8">
          <label for="title" class="block text-sm font-medium text-gray-700 mb-2">
            文章标题
          </label>
          <div class="mt-1 flex flex-col sm:flex-row shadow-sm rounded-md">
            <div class="relative flex-1">
              <input
                type="text"
                name="title"
                id="title"
                v-model="title"
                @keyup.enter="generateImage"
                class="focus:ring-indigo-500 focus:border-indigo-500 block w-full rounded-md sm:rounded-none sm:rounded-l-md sm:text-sm border-gray-300 p-3 sm:p-4 border text-base sm:text-lg mb-3 sm:mb-0 pr-10"
                placeholder="请输入文章标题，例如：2024年科技趋势..."
              />
              <div v-if="title" class="absolute inset-y-0 right-0 pr-3 flex items-center mb-3 sm:mb-0">
                <button 
                  @click="title = ''" 
                  class="text-gray-400 hover:text-gray-500 focus:outline-none"
                >
                  <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
                  </svg>
                </button>
              </div>
            </div>
            <button
              @click="generateImage"
              :disabled="loading || !title"
              class="inline-flex items-center justify-center px-6 py-3 border border-transparent text-base font-medium rounded-md sm:rounded-none sm:rounded-r-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors w-full sm:w-auto"
            >
              <svg v-if="loading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              {{ loading ? '生成中...' : '立即生成' }}
            </button>
          </div>
        </div>

        <!-- Progress Bar -->
        <div v-if="loading || (progress > 0 && progress < 100)" class="mb-8">
          <div class="relative pt-1">
            <div class="flex mb-2 items-center justify-between">
              <div>
                <span class="text-xs font-semibold inline-block py-1 px-2 uppercase rounded-full text-indigo-600 bg-indigo-100">
                  {{ progress < 100 ? '正在绘制...' : '完成' }}
                </span>
              </div>
              <div class="text-right">
                <span class="text-xs font-semibold inline-block text-indigo-600">
                  {{ progress }}%
                </span>
              </div>
            </div>
            <div class="overflow-hidden h-2 mb-4 text-xs flex rounded bg-indigo-100">
              <div
                :style="{ width: progress + '%' }"
                class="shadow-none flex flex-col text-center whitespace-nowrap text-white justify-center bg-indigo-500 transition-all duration-300 ease-out"
              ></div>
            </div>
          </div>
        </div>

        <!-- Error Message -->
        <div v-if="errorMsg" class="mb-8 bg-red-50 border-l-4 border-red-400 p-4 rounded-r-md animate-fade-in">
          <div class="flex">
            <div class="flex-shrink-0">
              <svg class="h-5 w-5 text-red-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
              </svg>
            </div>
            <div class="ml-3">
              <p class="text-sm text-red-700 font-medium">
                {{ errorMsg }}
              </p>
            </div>
          </div>
        </div>

        <!-- Result Image -->
        <div v-if="resultImage" class="mt-8 animate-fade-in">
          <div class="flex items-center justify-between mb-4">
             <h3 class="text-lg font-medium leading-6 text-gray-900">生成结果</h3>
             <button
              @click="downloadImage(resultImage)"
              class="inline-flex items-center px-3 py-1.5 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-colors cursor-pointer"
            >
              <svg class="-ml-1 mr-2 h-4 w-4 text-gray-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
              </svg>
              下载原图
            </button>
          </div>
         
          <div class="bg-gray-100 rounded-lg overflow-hidden shadow-lg ring-1 ring-black/5 group relative">
            <img 
              :src="resultImage" 
              alt="Generated Cover" 
              class="w-full h-auto object-cover transform transition-transform hover:scale-[1.01] duration-500 cursor-zoom-in"
              @click="showPreview = true"
            />
            <div 
              class="absolute inset-0 bg-black/0 group-hover:bg-black/10 transition-all duration-300 cursor-zoom-in flex items-center justify-center"
              @click="showPreview = true"
            >
              <span class="opacity-0 group-hover:opacity-100 bg-black/50 text-white px-4 py-2 rounded-full text-sm transition-opacity duration-300">
                点击预览大图
              </span>
            </div>
          </div>
        </div>
      </div>
      
      <div class="mt-8 text-center text-sm text-gray-500">
        有问题联系：243027571@qq.com
      </div>
    </div>

    <!-- Image Preview Modal -->
    <div 
      v-if="showPreview" 
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black bg-opacity-90 backdrop-blur-sm transition-opacity duration-300"
      @click="showPreview = false"
    >
      <div class="relative max-w-7xl max-h-full w-full flex items-center justify-center">
        <img 
          :src="resultImage" 
          alt="Full Preview" 
          class="max-w-full max-h-[90vh] object-contain rounded-lg shadow-2xl"
          @click.stop
        />
        <button 
          class="absolute top-4 right-4 text-white hover:text-gray-300 focus:outline-none"
          @click="showPreview = false"
        >
          <svg class="h-8 w-8" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<style>
@keyframes fade-in {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fade-in 0.5s ease-out forwards;
}
</style>
