<template>
  <div class="item">
    <div class="tit">{{ sort }}. 编辑器切换 ja 字体</div>
    <input type="checkbox" :checked="modelValue" @change="$emit('update:modelValue', $event.target.checked)" />
  </div>
</template>

<script>
import $ from "jquery";
export default {
  props: ["modelValue", "sort"],
  emits: ["update:modelValue"],
  data() {
    return {
      editorJaIntervalId: null // 添加变量存储定时器ID
    };
  },
  created() {
    if (this.modelValue) {
      this.editorJaIntervalId = setInterval(() => {
        if ($(".replyja").length < 1) {
          $("#reply-control .save-or-cancel .create").after(
            `<button class="btn btn-default replyja" style="margin-left:15px;" type="button">ja 字体</button>`
          );
          $(".replyja").click(function () {
            let $textarea = $(".d-editor-textarea-wrapper textarea");
            let text = `<span lang="ja">
${$(".d-editor-input").val()}
</span>`;

            $(".d-editor-textarea-wrapper textarea").val("");
            $textarea.focus(); // 聚焦到 textarea

            for (let i = 0; i < text.length; i++) {
              let char = text[i];

              // 更新 textarea 的值
              $textarea.val($textarea.val() + char);

              // 创建并派发 input 事件
              let inputEvent = new Event("input", {
                bubbles: true,
                cancelable: true,
              });
              $textarea[0].dispatchEvent(inputEvent);

              // 创建并派发 keydown 事件
              let keyEvent = new KeyboardEvent("keydown", {
                key: char,
                char: char,
                keyCode: char.charCodeAt(0),
                which: char.charCodeAt(0),
                bubbles: true,
              });
              $textarea[0].dispatchEvent(keyEvent);
            }
          });
        }
      }, 1000);
    }
  },
  beforeUnmount() {
    // 清除定时器
    if (this.editorJaIntervalId) {
      clearInterval(this.editorJaIntervalId);
    }
  },
  // Vue 2 兼容性
  beforeDestroy() {
    // 清除定时器
    if (this.editorJaIntervalId) {
      clearInterval(this.editorJaIntervalId);
    }
  }
};
</script>
