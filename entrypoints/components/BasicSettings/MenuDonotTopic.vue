<template>
  <div class="item">
    <div class="tit">
      {{ sort }}. 列表快速免打扰帖子
      <a href="https://linux.do/latest?state=muted" style="color: #e00; margin-left: 10px"
        >&lt;免打扰列表&gt;</a
      >
    </div>
    <input
      type="checkbox"
      :checked="modelValue"
      @change="$emit('update:modelValue', $event.target.checked)"
    />
  </div>
</template>

<script>
import $ from "jquery";
export default {
  props: ["modelValue", "sort"],
  emits: ["update:modelValue"],
  data() {
    return {
      donotTopicIntervalId: null // 添加变量存储定时器ID
    };
  },
  methods: {
    // 提示组件
    messageToast(message) {
      const messageElement = document.createElement("div");
      messageElement.className = "messageToast-text";
      messageElement.innerText = message;
      document.getElementById("messageToast").appendChild(messageElement);
      setTimeout(() => {
        messageElement.remove();
      }, 3000);
    },
    init() {
      if (window.location.href != "https://linux.do/latest?state=muted") {
        $(".topic-list .main-link a.title").each(function () {
          const id = $(this).attr("data-topic-id");
          if ($(this).parents(".link-top-line").find(".donottopic-btn").length < 1) {
            $(this)
              .parents(".link-top-line")
              .append(
                `<button class="btn btn-icon-text btn-default donottopic-btn donottopic-${id}" data-id="${id}">免打扰</button>`
              );
          }
        });
      } else {
        $(".topic-list .main-link a.title").each(function () {
          const id = $(this).attr("data-topic-id");
          if (
            $(this).parents(".link-top-line").find(".removedonottopic-btn").length < 1
          ) {
            $(this)
              .parents(".link-top-line")
              .append(
                `<button class="btn btn-icon-text btn-default removedonottopic-btn removedonottopic-${id}" data-id="${id}">移出免打扰</button>`
              );
          }
        });
      }

      function getCSRFToken() {
        return document.querySelector('meta[name="csrf-token"]').getAttribute("content");
      }
      let vm = this;
      // 点击免打扰
      $(document)
        .off("click", ".donottopic-btn")
        .on("click", ".donottopic-btn", function () {
          const formData = new FormData();
          formData.append("notification_level", 0);
          const topicId = $(this).attr("data-id");

          return new Promise((resolve, reject) => {
            // 发送带有 CSRF Token 的 POST 请求
            fetch(`https://linux.do/t/${topicId}/notifications`, {
              method: "POST",
              body: formData,
              headers: {
                "X-CSRF-Token": getCSRFToken(), // CSRF 令牌
                "X-Requested-With": "XMLHttpRequest", // 表明这是 XMLHttpRequest 请求
              },
              credentials: "include", // 附带 cookie
            })
              .then((response) => {
                if (!response.ok) {
                  reject(`HTTP error! status: ${response.status}`);
                }
                return response.json();
              })
              .then(() => {
                $(this).parents(".topic-list-item").remove();
                vm.messageToast("帖子已设为免打扰！");
                resolve();
              })
              .catch((error) => {
                console.log(error);
              });
          });
        });

      // 点击移出免打扰
      $(document)
        .off("click", ".removedonottopic-btn")
        .on("click", ".removedonottopic-btn", function () {
          const formData = new FormData();
          formData.append("notification_level", 1);
          const topicId = $(this).attr("data-id");

          return new Promise((resolve, reject) => {
            // 发送带有 CSRF Token 的 POST 请求
            fetch(`https://linux.do/t/${topicId}/notifications`, {
              method: "POST",
              body: formData,
              headers: {
                "X-CSRF-Token": getCSRFToken(), // CSRF 令牌
                "X-Requested-With": "XMLHttpRequest", // 表明这是 XMLHttpRequest 请求
              },
              credentials: "include", // 附带 cookie
            })
              .then((response) => {
                if (!response.ok) {
                  reject(`HTTP error! status: ${response.status}`);
                }
                return response.json();
              })
              .then(() => {
                $(this).parents(".topic-list-item").remove();
                vm.messageToast("帖子已移出免打扰！");
                resolve();
              })
              .catch((error) => {
                console.log(error);
              });
          });
        });
    },
  },
  created() {
    if (this.modelValue) {
      this.donotTopicIntervalId = setInterval(() => {
        this.init();
      }, 1000);
    }
  },
  beforeUnmount() {
    // 清除定时器
    if (this.donotTopicIntervalId) {
      clearInterval(this.donotTopicIntervalId);
    }
  },
  // Vue 2 兼容性
  beforeDestroy() {
    // 清除定时器
    if (this.donotTopicIntervalId) {
      clearInterval(this.donotTopicIntervalId);
    }
  }
};
</script>
