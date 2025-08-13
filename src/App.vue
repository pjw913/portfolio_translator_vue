<script>
import { ref } from 'vue';

export default{
  name:'App',
  setup(){
    const inputText = ref('');
    const outputText = ref('');
    const isKoreanToEnglish = ref(true);
    const error = ref('');
    const isTranslating = ref(false);

    const GOOGLE_API_KEY = 'AIzaSyBO3-_gzFfGdNX5tm9j0ZtxBzzVW1huUlw';

    const translateWithGoogle = async (text, sourceLang, targetLang) =>{
      try{
        const response = await fetch(
          `https://translation.googleapis.com/language/translate/v2?key=${GOOGLE_API_KEY}&q=${encodeURIComponent(text)}&source=${sourceLang}&target=${targetLang}`
        );
        if(!response.ok){
          const errorData = await response.json();
          console.error('Google Translation API error : ', errorData);
          throw new Error(`Google Translate API error:${response.status} - ${response.statusText}`)
        }
        const data = await response.json();
        console.log( data );
        return data.data.translations[0].translatedText;
      }catch (error){
        throw new Error(`Google Translate Error: ${error.message}`);
      }
    }
    const toggleDirection = () =>{
      isKoreanToEnglish.value = !isKoreanToEnglish.value;
      inputText.value = outputText.value;
      outputText.value='';
      error.value='';
    }
    const handleTranslate = async () =>{
      if(!inputText.value.trim()){
        error.value = 'enter text to translate'
        return
      }
      error.value=''
      isTranslating.value = true;
      try{
        const sourceLang = isKoreanToEnglish.value ? 'ko' : 'en';
        const targetLang = isKoreanToEnglish.value ? 'en' : 'ko';
        const translatedText = await translateWithGoogle(inputText.value, sourceLang, targetLang);
        outputText.value = translatedText;
      } catch (err){
        error.value = err.message;
      }finally{
        isTranslating.value = false
      }
    }
    return{
      inputText,
      outputText,
      isKoreanToEnglish,
      error,
      isTranslating,
      toggleDirection,
      handleTranslate
    }
  }  
}
</script>

<template>
  <div class="translator-container">
    <h1 class="title">
      {{  isKoreanToEnglish ? 'Korean to English' : 'English to Korean' }}
    </h1>
    <div class="input-section">
      <label for="inputText" class="label"> Input Text : </label>
      <textarea id="inputText" v-model="inputText" class="textarea" placeholder="Enter text to translate.." />
    </div>
    <button @click="toggleDirection" class="toggle-button" title="Switch translation direction">
      <span class="arrow-icon">↕</span>
    </button>
    <div class="output-section">
      <label for="outputText" class="label"> Translation : </label>
      <textarea id="outputText" v-model="outputText" readonly class="textarea" placeholder="Translation will ..." />
    </div>
    <div v-if="error" class="error-message">{{  error }}</div>
    <button @click="handleTranslate" class="translate-button" :disabled="isTranslating">
      {{  isTranslating ? 'Translating...' : 'Translate' }}
    </button>
  </div>
</template>

<style scoped>
  .translator-container{
    background-color: #fff;  padding:1.5rem;  border-radius:0.5rem;  width:100%;  max-width:28rem;
    box-shadow:0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06);  margin:0 auto;
  }
  .title{ font-size:1.5rem;  font-weight:bold;  margin-bottom:1rem; color:#1f2937;  }
  .input-section, .output-section{  margin-bottom:1rem;  }
  .label{  display:block;  font-weight:500;  margin-bottom:0.5rem;  color:#374151;  }
  .textarea{
    width:100%; height:8rem; padding:0.5rem; box-sizing:border-box; border:1px solid #d1d5db;
    border-radius:0.5rem; font-family: inherit;  resize:vertical;
  }
  .textarea:focus{ outline:none;  border-color:#3b82f6; box-shadow:0 0 0 1px #3b82f6;  }
  .textarea:read-only{  background-color:#f9fafb;  color:#6b7280;  }
  .toggle-button{
    background-color:#3b82f6; color:#fff; font-weight:500; padding:0.5rem 1rem; border:none;
    border-radius:0.5rem; cursor: pointer; margin-bottom:1rem;
    display:flex; justify-content: center;  align-items: center;  transition:background-color 0.2s;
  }
  .toggle-button:hover{  background-color:#2563eb;  }
  .toggle-button:active{  background-color:#1d4ed8;  }
  .arrow-icon{  font-size:1.25rem;  font-weight:bold;   }
  .error-message{
    background:#f3f2f2;  border:1px solid #fecaca;  color:#dc2626;  padding:0.75rem 1rem;
    border-radius:0.375rem;  margin-bottom: 1rem;  font-size: 0.875rem;
  }
  .translate-button{
    background-color:#3b82f6;  color:#fff;  font-weight: 500;  padding: 0.5rem 1rem;  cursor:pointer;
    border-radius: 0.5rem;  border:0 none;  width:100%;  transition:background-color 0.2s;
  }
  .translate-button:hover:not(:disabled){  background-color:#2563eb;  }
  .translate-button:active:not(:disabled){  background-color:#1d4ed8; }
  .translate-button:disabled{  background-color:#9ca3af;  cursor:not-allowed;  }
</style>
