<template>
  <div class="wedding-editor" ref="editor">
    <header class="editor-header">
        <a href="javascript:;"></a>
        <a href="javascript:;" class="minimum"></a>
        <a href="javascript:;" class="maximum"></a>
    </header>
    <!-- 日期 -->
    <p class="code">Last login: <span>{{ startDate }}</span> on ttys001</p>
    <!--代码编辑区-->
    <pre>
      <code v-html="highlightedCode"></code>
    </pre>
    <Executions :canExecute="canExecute" @onUpdating="scrollToBottom" @onFinish="canOpen = true"/>
    <Invitation :canOpen="canOpen" @sendBarrage="onAfterSending"/>
    <Barrage :wish="wish" :canStart="canStart"/>
  </div>
</template>

<script>
  import Prism from 'prismjs'
  import 'prismjs/themes/prism-okaidia.css'
  import '../utils/raf'
  import data from '../mock/data'

  import Executions from './Executions'
  import Invitation from './Invitation'
  import Barrage from './Barrage'

  export default {
    name: 'Editor',
    components: { Executions, Invitation, Barrage },
    data() {
      return {
        startDate: '',
        code: data.code,
        currentCode: '',
        isCursorVisible: 1,
        canExecute: false,
        canOpen: false,
        wish: '',
        canStart: false
      }
    },
    created() {
      this.startDate = (new Date()).toDateString()
      this.progressivelyTyping()
    },
    updated() {
      this.scrollToBottom()
    },
    computed: {
      highlightedCode () {
        const code = Prism.highlight(
            this.currentCode,
            Prism.languages.javascript
          )
        const codeWithCursor = `${code}<span style="opacity:${this.isCursorVisible}"> ▍</span>`
        return codeWithCursor
      }
    },
    methods: {
      scrollToBottom() {
        // Keep the page rolling all the way down to the bottom
        this.$refs.editor.scrollTop = 100000
      },
      // Code input
      progressivelyTyping() {
        return new Promise((resolve) => {
          let count = 0, typingCount = 0, typing
          // Write code for each frame function
          let step = () => {
            let randomNumber = Math.round(Math.random() * 6)
            // Touch the random speed of your typing
            if(count % 2 === 0 && randomNumber % 4 === 0){
              this.currentCode = this.code.substring(0, typingCount)
              typingCount++
            }
            // The cursor flashes approximately every 24 frames
            if(count % 24 === 0){
              this.isCursorVisible = this.isCursorVisible === 0 ? 1 : 0
            }
            count++
            if (typingCount <= this.code.length) {
              typing = requestAnimationFrame(step)
            } else {
              resolve()
              this.canExecute = true
              cancelAnimationFrame(typing)
            }
          }
          typing = requestAnimationFrame(step)
        })
      },
      // After sending the bullet
      onAfterSending(wish) {
        this.wish = wish
        this.canOpen = false
        setTimeout(() => {
          this.canStart = true
        }, 800)
      }
    }
  }
</script>

<style lang="less">
.wedding-editor{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 15px;
  padding-top: 50px;
  overflow-x: hidden;
  overflow-y: auto;
  z-index: 1;
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  -webkit-transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  .editor-header{
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    padding: 12px;
    overflow: hidden;
    background: #2B2B48;
    z-index: 3;
    >a{
      float: left;
      display: block;
      width: 16px;
      height: 16px;
      margin-right: 5px;
      border-radius: 8px;
      background: #FC615D;
      &.minimum{
        background: #FDBC40;
      }
      &.maximum{
        background: #34C84A;
      }
    }
  }
  p.code{
    margin: 0;
    color: #BBB;
    line-height: 1.2;
    font-family: 'Roboto Mono', 'Menlo', 'Monaco', Courier, monospace !important;
    font-weight: 500 !important;
    font-size: 16px!important;
  }
  pre{
    margin: 0;
    white-space: pre-wrap;
    code{
      white-space: pre-wrap;
      word-break: break-all;
      font-size: 16px!important;
      margin: 0;
      color: #BBB;
      line-height: 1.2;
      font-family: 'Roboto Mono', 'Menlo', 'Monaco', Courier, monospace !important;
      font-weight: 500 !important;
      background: transparent;
    }
  }
}
</style>