<template>
  <div id="app">
    <h1>PDF预览例子</h1>
    <p>
      <button @click="pdf_url='东风日产供应商购车指南.pdf'">默认文档</button>
      <button @click="pdf_url='pdf样本/1599126417040.pdf'">pdf样本/1599126417040.pdf</button>
      <button @click="pdf_url='pdf样本/转换不了.pdf'">pdf样本/转换不了.pdf</button>
      <button @click="pdf_url='pdf样本/最大.pdf'">pdf样本/最大.pdf</button>
    </p>
    <PDFViewer :url="pdf_url" />
  </div>
</template>

<script>
import PDFViewer from "./components/PDFViewer"
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data(){
    return {
      pdf_url: "东风日产供应商购车指南.pdf"
    }
  },
  components: {
    HelloWorld,
    PDFViewer
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
