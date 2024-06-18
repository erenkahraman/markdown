<template>
    <div ref="content"></div>
  </template>
  
  <script>
  export default {
    name: 'MathJax',
    props: {
      content: {
        type: String,
        required: true
      }
    },
    mounted() {
      this.renderMath();
    },
    updated() {
      this.renderMath();
    },
    methods: {
      renderMath() {
        this.$refs.content.innerHTML = this.content;
        window.MathJax.typesetPromise([this.$refs.content]);
      }
    }
  };
  </script>