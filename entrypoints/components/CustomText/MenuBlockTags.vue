<template>
  <div class="item">
    <div class="tit">
      {{ sort }}. 标签屏蔽功能（使用英文，分隔）屏蔽指定标签
    </div>
  </div>
  <textarea v-model="textarea" @input="handleChange"> </textarea>
</template>

<script>
import $ from "jquery";
export default {
  props: {
    value: {
      type: String,
      default: "",
    },
    sort: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      textarea: this.value,
    };
  },
  watch: {
    value(newValue) {
      this.textarea = newValue;
    },
  },
  methods: {
    handleChange() {
      this.$emit("update:value", this.textarea);
    },
    init() {
      if (!this.textarea) return;

      // 将用户输入的关键字分割为数组，并去除空格和空值
      const keywords = this.textarea
        .split(",")
        .map((keyword) => keyword.trim())
        .filter(Boolean);
      if (keywords.length === 0) return;

      // 安全检查 jQuery 的使用，防止元素不可用时出错
      try {
        // 检查话题标签
        $(".topic-list-item .discourse-tags a")
          .filter((index, element) => {
            const text = $(element).text();
            return keywords.some((keyword) => text.includes(keyword));
          })
          .parents("tr.topic-list-item")
          .remove();
      } catch (error) {
        console.error("init 方法出错：", error);
      }
    },
  },
  created() {
    if (this.textarea) {
      let previousTopicListLength = 0;
      let previousPostStreamLength = 0;

      this.pollingInterval = setInterval(() => {
        try {
          const currentTopicListLength = $(".topic-list-body tr").length || 0;
          const currentPostStreamLength = $(".post-stream .topic-post").length || 0;

          if (previousTopicListLength !== currentTopicListLength) {
            previousTopicListLength = currentTopicListLength;
            this.init();
          }
          if (previousPostStreamLength !== currentPostStreamLength) {
            previousPostStreamLength = currentPostStreamLength;
            this.init();
          }
        } catch (error) {
          console.error("轮询逻辑出错：", error);
        }
      }, 1000);
    }
  },
  beforeDestroy() {
    // 在组件销毁前清除定时器，防止内存泄漏
    if (this.pollingInterval) {
      clearInterval(this.pollingInterval);
    }
  },
};
</script>

<style lang="less" scoped>
.item {
  border: none !important;
}
</style>
