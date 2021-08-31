<template>
  <!--视频窗口展示-->
  <div :id="id"></div>
</template>
<script>
export default {
  name: "hkVideo",
  data() {
    return {
      // 视频播放
      initCount: 0, // 初始化重试次数,3次失败则报错
      oWebControl: null, // 页面控制对象
      pubKey: "",
      cameraIp: "xxx.xxx.xx.xx", // 综合安防管理平台IP地址
    };
  },
  props: ["code", "id"], // 接收父页面传来的摄像头 code
  methods: {
    // 创建播放实例
    initPlugin() {
      // 初始化播放插件
      const _this = this;
      _this.oWebControl = new WebControl({
        szPluginContainer: _this.id, // 指定容器id
        iServicePortStart: 15900, // 指定起止端口号，建议使用该值
        iServicePortEnd: 15909,
        szClassId: "23BF3B0A-2C56-4D97-9C03-0CB103AA8F11", // 用于IE10使用ActiveX的clsid
        cbConnectSuccess: function () {
          // 创建WebControl实例成功
          console.log("创建WebControl实例成功");
          _this.oWebControl
            .JS_StartService("window", {
              // WebControl实例创建成功后需要启动服务
              dllPath: "./VideoPluginConnect.dll", // 值"./VideoPluginConnect.dll"写死
            })
            .then(
              function () {
                // 启动插件服务成功
                console.log("启动插件服务成功");
                _this.oWebControl.JS_SetWindowControlCallback({
                  // 设置消息回调
                  cbIntegrationCallBack: _this.cbIntegrationCallBack,
                });
                let divWidth = document.getElementById(_this.id).scrollWidth;
                let divHeight = document.getElementById(_this.id).scrollHeight;
                _this.oWebControl
                  .JS_CreateWnd(_this.id, divWidth, divHeight)
                  .then(function () {
                    //JS_CreateWnd创建视频播放窗口，宽高可设定
                    _this.init(); // 创建播放实例成功后初始化
                  });
              },
              function () {
                // 启动插件服务失败
              }
            );
        },
        cbConnectError: function () {
          // 创建WebControl实例失败
          _this.oWebControl = null;
          console.log("warn,插件未启动，正在尝试启动，请稍候...");
          WebControl.JS_WakeUp("VideoWebPlugin://"); // 程序未启动时执行error函数，采用wakeup来启动程序
          initCount++;
          if (initCount < 3) {
            setTimeout(function () {
              _this.initPlugin();
            }, 3000);
          } else {
            console.log("error, 插件启动失败，请检查插件是否安装!");
          }
        },
        cbConnectClose: function (bNormalClose) {
          // 异常断开：bNormalClose = false
          // JS_Disconnect正常断开：bNormalClose = true
          if (!bNormalClose) {
            console.log("error, 视屏链接异常中断！");
          }
          _this.oWebControl = null;
        },
      });
    },

    //初始化, 主要改这里
    init() {
      const _this = this;
      this.getPubKey(function () {
        // 请自行修改以下变量值
        let appkey = "xxxxxxx"; //综合安防管理平台提供的appkey，必填
        let secret = _this.setEncrypt("xxxxxxxxxx"); //综合安防管理平台提供的secret，必填
        let ip = _this.cameraIp; //综合安防管理平台IP地址，必填
        let playMode = 0; //初始播放模式：0-预览，1-回放
        let port = 8443; //综合安防管理平台端口，若启用HTTPS协议，默认443
        let snapDir = "D:\\SnapDir"; //抓图存储路径
        let videoDir = "D:\\VideoDir"; //紧急录像或录像剪辑存储路径
        let layout = "1x1"; //playMode指定模式的布局
        let enableHTTPS = 1; //是否启用HTTPS协议与综合安防管理平台交互，这里总是填1
        let encryptedFields = "secret"; //加密字段，默认加密领域为secret
        let showToolbar = 0; //是否显示工具栏，0-不显示，非0-显示
        let showSmart = 1; //是否显示智能信息（如配置移动侦测后画面上的线框），0-不显示，非0-显示
        let buttonIDs = ""; //自定义工具条按钮
        // 请自行修改以上变量值

        _this.oWebControl
          .JS_RequestInterface({
            funcName: "init",
            argument: JSON.stringify({
              appkey: appkey, //API网关提供的appkey
              secret: secret, //API网关提供的secret
              ip: ip, //API网关IP地址
              playMode: playMode, //播放模式（决定显示预览还是回放界面）
              port: port, //端口
              snapDir: snapDir, //抓图存储路径
              videoDir: videoDir, //紧急录像或录像剪辑存储路径
              layout: layout, //布局
              enableHTTPS: enableHTTPS, //是否启用HTTPS协议
              encryptedFields: encryptedFields, //加密字段
              showToolbar: showToolbar, //是否显示工具栏
              showSmart: showSmart, //是否显示智能信息
              buttonIDs: buttonIDs, //自定义工具条按钮
            }),
          })
          .then((oData) => {
            let divWidth = document.getElementById(_this.id).scrollWidth;
            let divHeight = document.getElementById(_this.id).scrollHeight;
            _this.oWebControl.JS_Resize(divWidth, divHeight); // 初始化后resize一次，规避firefox下首次显示窗口后插件窗口未与DIV窗口重合问题
            _this.startPreview();
          });
      });
    },

    //获取公钥, 不用改
    getPubKey(callback) {
      this.oWebControl
        .JS_RequestInterface({
          funcName: "getRSAPubKey",
          argument: JSON.stringify({
            keyLength: 1024,
          }),
        })
        .then((oData) => {
          if (oData.responseMsg.data) {
            this.pubKey = oData.responseMsg.data;
            callback();
          }
        });
    },

    //RSA加密, 不用改
    setEncrypt(value) {
      let encrypt = new JSEncrypt();
      encrypt.setPublicKey(this.pubKey);
      return encrypt.encrypt(value);
    },

    //视频预览功能, 就设置 cameraIndexCode 就行了
    startPreview() {
      let _this = this;
      let cameraIndexCode = _this.code; //获取输入的监控点编号值，必填
      let streamMode = 0; //主子码流标识：0-主码流，1-子码流
      let transMode = 1; //传输协议：0-UDP，1-TCP
      let gpuMode = 0; //是否启用GPU硬解，0-不启用，1-启用
      let wndId = -1; //播放窗口序号（在2x2以上布局下可指定播放窗口）

      cameraIndexCode = cameraIndexCode.replace(/(^\s*)/g, "");
      cameraIndexCode = cameraIndexCode.replace(/(\s*$)/g, "");

      _this.oWebControl.JS_RequestInterface({
        funcName: "startPreview",
        argument: JSON.stringify({
          cameraIndexCode: cameraIndexCode, //监控点编号
          streamMode: streamMode, //主子码流标识
          transMode: transMode, //传输协议
          gpuMode: gpuMode, //是否开启GPU硬解
          wndId: wndId, //可指定播放窗口
        }),
      });
    },
    //停止全部预览
    stopAlldwPreview() {
      this.oWebControl.JS_RequestInterface({
        funcName: "stopAllPreview",
      });
    },
    destroyedView() {
      if (this.oWebControl != null) {
        this.stopAlldwPreview();
        this.oWebControl.JS_HideWnd(); // 先让窗口隐藏，规避可能的插件窗口滞后于浏览器消失问题
        this.oWebControl.JS_Disconnect().then(
          function () {
            // 断开与插件服务连接成功
          },
          function () {
            // 断开与插件服务连接失败
          }
        );
      }
    },
  },
  mounted() {
    // 有摄像头 code, 才加载, 另外因为我每次只显示一个, 所以显示之前要把之前显示的摄像头销毁, 所以加了 this.oWebControl == null 的判断.
    if (this.cameraIndexCode && this.oWebControl == null) {
      this.initPlugin();
    } else {
      // 如果 code 不为空, 则先销毁现有摄像头, 再去加载新的摄像头
      this.destroyedView();
      setTimeout(this.initPlugin(), 1000);
    }
  },
  destroyed() {
    this.destroyedView();
  },
};
</script>
