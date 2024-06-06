<template>
    <div class="wrapper">
        
        <div class="left-col">
            <h3>Question Markdown</h3>
            <textarea v-model="userInput" />
        </div>

        <div class="right-col">
            <h3>Raw</h3>
            <div v-html="rawMarkdown"></div>
            <hr>
            <h3>Sanitized with DOMPurify</h3>
            <div v-html="sanitizedMarkdown"></div>
        </div>

    </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { marked } from 'marked'
import DOMPurify from 'DOMPurify'

const userInput = ref(`#### Sanitize Example \n\n Below is a simple, malicious example of html in markdown that we catch with DOM Purify before injecting the html output of 'marked.' \n\n<div>\n  <button onclick="alert('This is an XSS attack!')">\n    Click me\n  </button>\n</div>` )
const rawMarkdown = computed(() => marked.parse(userInput.value))
const sanitizedMarkdown = computed(() => DOMPurify.sanitize(rawMarkdown.value))


</script>

<style>
.wrapper {
    display: flex;
    height: 100vh;
    width: 100%;
    justify-content: center;
}
.left-col, .right-col {
    width: 50%;
    max-width: 400px;
    height: 100%;
    padding: 8px;
}
textarea {
    width: 100%;
    height: 300px;
}
</style>
