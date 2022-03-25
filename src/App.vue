<template>
  <div id="app">
    <div id="pdf">
      <template v-if="loading">
        <div class="loading-box">
          <div class="icon-box">
            <div class="folder">
              <span class="folder-tab"></span>
              <div class="folder-icn">
                <div class="downloading">
                  <span class="custom-arrow"></span>
                </div>
                <div class="bar"></div>
              </div>
            </div>
          </div>
          <p class="loading-info">文件正在加载中...</p>
        </div>
      </template>
    </div>
  </div>
</template>
<script>
import Pdfh5 from "pdfh5";
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      isReadComplete: false, // 是否阅读完成
      pdfh5: null,
      loading: true,
    };
  },
  mounted() {
    this.initData();
  },
  methods: {
    initData() {
      this.listenReady();
      this.loadPdf();
    },
    loadPdf() {
      const url = this.getQueryVariable("url");
      const file = url || "https://www.gjtool.cn/pdfh5/git.pdf";
      axios
        .get(file, {
          responseType: "arraybuffer",
        })
        .then((res) => {
          this.pdfh5 = new Pdfh5("#pdf", {
            data: res.data,
          });
          this.$nextTick(() => {
            this.bindEvent();
          });
        });
    },
    // 绑定事件
    bindEvent() {
      const _this = this;
      const pdfh5 = this.pdfh5;
      // pdf加载完成
      pdfh5.on("complete", function (status, msg, time) {
        _this.loading = false;
        _this.postMessage("complete", {
          status,
          msg,
          time,
          totalNum: this.totalNum,
        });
        if(this.totalNum < 3){
          // _this.postMessage("readComplete", { type: 2 });
        }
      });
      // pdf滚动
      pdfh5.on("scroll", function (scrollTop) {
        const currentNum = pdfh5.currentNum;
        const totalNum = pdfh5.totalNum;
        if (currentNum === totalNum && !_this.isReadComplete) {
          _this.isReadComplete = true;
          _this.postMessage("readComplete", { scrollTop: scrollTop, totalNum: this.totalNum });
        }
      });
    },
    listenReady() {
      document.addEventListener("UniAppJSBridgeReady", function () {
        uni.getEnv(function (res) {
          console.log("当前环境：" + JSON.stringify(res));
        });
      });
    },
    getQueryVariable(variable) {
      var query = window.location.search.substring(1);
      var vars = query.split("&");
      for (var i = 0; i < vars.length; i++) {
        var pair = vars[i].split("=");
        if (pair[0] == variable) {
          return pair[1];
        }
      }
      return false;
    },
    postMessage(type, data) {
      console.log("发送消息", type, data);
      uni.postMessage({
        data: {
          action: type,
          data: data,
        },
      });
    },
  },
};
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
}
html,
body,
#app,
#pdf {
  width: 100%;
  height: 100%;
  background: #f3f3f3;
}

// Here is where the magic happens

$offset: 187;
$duration: 1.4s;

.loading-box {
  position: fixed;
  width: 100%;
  left: 0;
  left: 0;
  top: calc(50% - 52px);
  text-align: center;
  .loading-info {
    height: 60px;
    line-height: 60px;
    text-align: center;
    color: #666;
    font-size: 13px;
  }

  .icon-box {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.folder {
  background-color: #b8dbf8;
  position: relative;
  width: 92px;
  height: 64px;
  display: block;
  border-top-right-radius: 8px;
  border-bottom-right-radius: 8px;
  border-bottom-left-radius: 8px;
}
.folder-tab {
  position: absolute;
  height: 10px;
  left: 0;
  bottom: 100%;
  display: block;
  width: 40%;
  border-top-left-radius: 8px;
  background-color: inherit;
  &:after {
    content: "";
    position: absolute;
    display: block;
    top: 0;
    left: calc(100% - 10px);
    border-bottom: 10px solid #cce7fd;
    border-left: 10px solid transparent;
    border-right: 10px solid transparent;
  }
}

.folder-icn {
  padding-top: 12px;
  width: 100%;
  height: 100%;
  display: block;
}
.downloading {
  width: 30px;
  height: 32px;
  margin: 0 auto;
  position: relative;
  overflow: hidden;
}
.custom-arrow {
  width: 14px;
  height: 14px;
  position: absolute;
  top: 0;
  left: 50%;
  margin-left: -7px;
  background-color: #fff;

  -webkit-animation-name: downloading;
  -webkit-animation-duration: 1.5s;
  -webkit-animation-iteration-count: infinite;
  animation-name: downloading;
  animation-duration: 1.5s;
  animation-iteration-count: infinite;

  &:after {
    content: "";
    position: absolute;
    display: block;
    top: 100%;
    left: -9px;
    border-top: 15px solid #fff;
    border-left: 16px solid transparent;
    border-right: 16px solid transparent;
  }
}
.bar {
  width: 30px;
  height: 4px;
  background-color: #fff;
  margin: 0 auto;
}

@-webkit-keyframes downloading {
  0% {
    top: 0;
    opacity: 1;
  }
  50% {
    top: 110%;
    opacity: 0;
  }
  52% {
    top: -110%;
    opacity: 0;
  }
  100% {
    top: 0;
    opacity: 1;
  }
}
@keyframes downloading {
  0% {
    top: 0;
    opacity: 1;
  }
  50% {
    top: 110%;
    opacity: 0;
  }
  52% {
    top: -110%;
    opacity: 0;
  }
  100% {
    top: 0;
    opacity: 1;
  }
}
</style>
