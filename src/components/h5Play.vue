<template>
  <div :id="id"></div>
</template>
<script>
export default {
  data() {
    return {
      oPlugin: null,
      iWind: 0,
      curIndex: 0,
    };
  },
  props: ["id", "code"],
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.oPlugin = new JSPlugin({
        szId: this.id,
        szBasePath: "./",
        oStyle: {
          border: "#343434",
          borderSelect: "#FFCC00",
          background: "#000",
        },
        openDebug: true,
      });
      this.initPlugin();
    },
    initPlugin() {
      this.oPlugin.JS_SetWindowControlCallback({
        windowEventSelect: function (iWndIndex) {
          //插件选中窗口回调
          this.iWind = iWndIndex;
          console.log(iWndIndex);
        },
        pluginErrorHandler: function (iWndIndex, iErrorCode, oError) {
          //插件错误回调
          console.error(
            `window-${iWndIndex}, errorCode: ${iErrorCode}`,
            oError
          );
        },
        windowEventOver: function (iWndIndex) {
          //鼠标移过回调
          //console.log(iWndIndex);
        },
        windowEventOut: function (iWndIndex) {
          //鼠标移出回调
          //console.log(iWndIndex);
        },
        windowEventUp: function (iWndIndex) {
          //鼠标mouseup事件回调
          //console.log(iWndIndex);
        },
        windowFullCcreenChange: function (bFull) {
          //全屏切换回调
          console.log(bFull);
        },
        firstFrameDisplay: function (iWndIndex, iWidth, iHeight) {
          //首帧显示回调
          console.log(iWndIndex, iWidth, iHeight);
        },
        performanceLack: function () {
          //性能不足回调
          console.log("ddiwduw");
        },
      });
      this.oPlugin
        .JS_SetOptions({
          // bSupportSound: false  //是否支持音频，默认支持
          // bSupporDoubleClickFull: false  //是否双击窗口全屏，默认支持
          // bOnlySupportMSE: true  //只支持MSE
          // bOnlySupportJSDecoder: true  //只支持JSDecoder
        })
        .then(() => {
          // 这里需要请求后端地址 最好为ws流.
          this.realplay();
        });
    },
    realplay() {
      let url = "ws://xxxxxxxxxxxxxxx";
      this.oPlugin
        .JS_Play(
          url,
          {
            playURL: url,
            mode: 1,
          },
          this.curIndex
        )
        .then(
          function () {
            console.log("realplay success");
          },
          function () {
            console.log("realplay failed");
          }
        );
    },
    resizePlay(index) {
      this.oPlugin.JS_Resize(500, 300).then(
        () => {
          console.info("JS_Resize success");
          // do you want...
        },
        (err) => {
          console.info("JS_Resize failed");
          // do you want...
        }
      );
    },
  },
};
</script>