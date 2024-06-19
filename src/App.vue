<template>
    <div class="wrapper">
      <div class="left-col">
        <h3>Markdown Question Editor</h3>
        <div class="editor-container">
          <textarea
            v-model="newQuestion"
            @dragover.prevent
            @drop="handleDrop"
            placeholder="Enter your question in Markdown format..."
          ></textarea>
          <div class="math-symbols">
            <h4>Math Symbols</h4>
            <div class="symbol" draggable="true" @dragstart="dragStart">√ (Square Root)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">x² (Exponent)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">log (Logarithm)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">∫ (Integral)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">∞ (Infinity)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">∑ (Summation)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">π (Pi)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">θ (Theta)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">Δ (Delta)</div>
            <div class="symbol" draggable="true" @dragstart="dragStart">α (Alpha)</div>
          </div>
        </div>
        <div class="media-library">
          <h4>Media Library</h4>
          <div class="media-row">
            <div
              v-for="(image, index) in mediaLibrary"
              :key="index"
              class="media-item"
              draggable="true"
              @dragstart="dragStartImage($event, image.url)"
            >
              <img :src="image.url" alt="Uploaded" />
            </div>
            <div class="media-item add-item" @click="triggerFileUpload">
              <span>+</span>
              <input type="file" ref="fileInput" @change="handleFileUpload" style="display: none;" />
            </div>
          </div>
        </div>
        <div v-if="selectedType === 'multiple'">
          <h4>Multiple Choice Options</h4>
          <div v-for="(option, optIndex) in newQuestionOptions" :key="optIndex">
            <label>{{ String.fromCharCode(65 + optIndex) }}. </label>
            <input
              v-model="newQuestionOptions[optIndex]"
              @input="handleOptionInput($event, optIndex)"
              placeholder="Option"
            />
            <input type="radio" :name="'correctOption' + questions.length" @click="handleOptionClick(optIndex)" />
          </div>
          <button @click="addMultipleChoiceOption">Add Option</button>
        </div>
        <div v-if="selectedType === 'fill-in'">
          <h4>Enter the correct answer:</h4>
          <input v-model="correctFillInAnswer" placeholder="Enter the correct answer..." />
        </div>
        <div class="button-group">
          <select v-model="selectedType" @change="chooseQuestionType">
            <option disabled value="">Choose question type</option>
            <option value="multiple">Multiple Choice</option>
            <option value="fill-in">Fill in the Blank</option>
          </select>
          <button @click="addQuestion">Create Item</button>
          <button @click="showPreview">Preview</button>
          <button @click="showGuidance = true">Markdown Guidance</button>
        </div>
      </div>
      <div class="right-col">
        <h3>Preview</h3>
        <div v-for="(question, index) in questions" :key="index">
          <div v-html="DOMPurify.sanitize(marked.parse(`${index + 1}. ${question.text}`))" class="preview"></div>
          <div v-if="question.type === 'multiple'">
            <div v-for="(option, optIndex) in question.options" :key="optIndex">
              <input 
                type="radio" 
                :name="'previewOption' + index" 
                :value="option" 
                v-model="questions[index].selectedOption"
              />
              <label>{{ String.fromCharCode(65 + optIndex) }}. {{ option }}</label>
            </div>
          </div>
          <div v-if="question.type === 'fill-in'">
            <h4>Fill in the Blank Answer:</h4>
            <input v-model="questions[index].answer" placeholder="Enter your answer here..." />
          </div>
        </div>
      </div>
      <div v-if="showGuidance" class="modal" @click.self="showGuidance = false">
        <div class="modal-content">
          <span class="close" @click="showGuidance = false">&times;</span>
          <h3>Markdown Guidance</h3>
          <p>You can find detailed Markdown syntax guide here: <a href="https://www.markdownguide.org/basic-syntax/" target="_blank">Markdown Guide</a></p>
        </div>
      </div>
      <div v-if="showPreviewModal" class="modal" @click.self="showPreviewModal = false">
        <div class="modal-content">
          <span class="close" @click="showPreviewModal = false">&times;</span>
          <h3>Preview</h3>
          <div v-for="(question, index) in questions" :key="index">
            <div v-html="DOMPurify.sanitize(marked.parse(`${index + 1}. ${question.text}`))" class="preview"></div>
            <div v-if="question.type === 'multiple'">
              <div v-for="(option, optIndex) in question.options" :key="optIndex">
                <input 
                  type="radio" 
                  :name="'previewOption' + index" 
                  :value="option" 
                  v-model="questions[index].selectedOption"
                />
                <label>{{ String.fromCharCode(65 + optIndex) }}. {{ option }}</label>
              </div>
            </div>
            <div v-if="question.type === 'fill-in'">
              <h4>Fill in the Blank Answer:</h4>
              <input v-model="question.answer" placeholder="Enter your answer here..." />
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>

<script setup>
import { ref, getCurrentInstance } from 'vue';
import { marked } from 'marked';
import DOMPurify from 'dompurify';
import defaultMultiple from './helpers/demo-questions/defaultMultiple.js';
import defaultFilling from './helpers/demo-questions/defaultFilling.js';

const { proxy } = getCurrentInstance();
const newQuestion = ref("");
const newQuestionOptions = ref(['']);
const questions = ref([]);
const showGuidance = ref(false);
const showPreviewModal = ref(false);
const correctFillInAnswer = ref('');

const selectedType = ref('');
const mediaLibrary = ref([]);

const dragStart = (event) => {
  const data = event.target.textContent;
  console.log('Dragging data:', data);
  event.dataTransfer.setData('text/plain', data);
};

const dragStartImage = (e, url) => {
    e.dataTransfer.setData('text', buildImgHtml('https://picsum.photos/200/300'));
    console.log(e, url);
    function buildImgHtml(url) {
        return `<img src="${url}" width="300px" height="300px">`
    }
};


const addQuestion = () => {
  if (newQuestion.value.trim() !== "") {
    questions.value.push({
      text: newQuestion.value,
      type: selectedType.value,
      options: [...newQuestionOptions.value],
      correctOption: selectedType.value === 'multiple' ? '' : correctFillInAnswer.value,
      answer: '',
      selectedOption: ''
    });
    newQuestion.value = '';
    newQuestionOptions.value = [''];
    correctFillInAnswer.value = '';
  }
};

const addMultipleChoiceOption = () => {
  newQuestionOptions.value.push('');
};

const handleOptionInput = (event, optionIndex) => {
  newQuestionOptions.value[optionIndex] = event.target.value;
};

const chooseQuestionType = () => {
  if (selectedType.value === 'multiple') {
    newQuestion.value = defaultMultiple;
    newQuestionOptions.value = ['Option A', 'Option B', 'Option C', 'Option D'];
  } else if (selectedType.value === 'fill-in') {
    newQuestion.value = defaultFilling;
    newQuestionOptions.value = [];
  }
};

const showPreview = () => {
  showPreviewModal.value = true;
};

const triggerFileUpload = () => {
  proxy.$refs.fileInput.click();
};

const handleFileUpload = (event) => {
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (e) => {
    mediaLibrary.value.push({ url: e.target.result });
  };
  reader.readAsDataURL(file);
};

const handleDrop = (event) => {
  event.preventDefault();
  const data = event.dataTransfer.getData('text/plain');
  newQuestion.value += data;
};

const handleOptionClick = (optIndex) => {
  const lastQuestionIndex = questions.value.length - 1;
  if (lastQuestionIndex >= 0 && questions.value[lastQuestionIndex]) {
    questions.value[lastQuestionIndex].correctOption = newQuestionOptions.value[optIndex];
  }
};

</script>

<style>
.wrapper {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px;
  gap: 20px;
  background: #f4f4f9;
  height: 100vh;
  box-sizing: border-box;
}

.left-col,
.right-col {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 10px;
  background: #ffffff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  height: 100%;
  overflow: auto;
}

.editor-container {
  display: flex;
  gap: 20px;
  width: 90%;
}

textarea {
  width: 70%;
  height: 400px;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  box-sizing: border-box;
  font-size: 16px;
  resize: none;
}

.math-symbols {
  width: 40%;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  background: #f9f9f9;
}

.math-symbols h4 {
  margin: 0;
  margin-bottom: 10px;
  font-size: 16px;
  color: #333;
}

.symbol {
  padding: 5px;
  margin: 5px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: #e9e9e9;
  cursor: grab;
  text-align: center;
}

.symbol:active {
  cursor: grabbing;
}

.media-library {
  width: 100%;
  margin-top: 10px;
}

.media-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.media-item {
  width: 100px;
  height: 100px;
  overflow: hidden;
  border: 1px solid #ddd;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: grab;
  position: relative;
}

.media-item img {
  max-width: 100%;
  max-height: 100%;
}

.media-item.add-item {
  cursor: pointer;
  border: 2px dashed #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
}

.media-item.add-item span {
  font-size: 24px;
  color: #ccc;
}

.media-item:active {
  cursor: grabbing;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.button-group button,
.button-group select {
  padding: 10px 20px;
  font-size: 14px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.button-group button {
  background: #007bff;
  color: #fff;
}

.button-group select {
  background: #fff;
  color: #333;
}

.button-group button:hover {
  background: #0056b3;
}

.preview {
  width: 100%;
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  overflow-y: auto;
  box-sizing: border-box;
}

.modal {
  display: flex;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 70%;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}
</style>