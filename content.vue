<template>
  <div id="Content">
    <el-dialog
      title="AI预测中"
      :visible.sync="dialogTableVisible"
      :show-close="false"
      :close-on-press-escape="false"
      :append-to-body="true"
      :close-on-click-modal="false"
      :center="true"
    >
      <el-progress :percentage="percentage"></el-progress>
      <span slot="footer" class="dialog-footer">请耐心等待约3秒钟</span>
    </el-dialog>

    <div id="CT">
      <div id="CT_image">
        <el-card
          id="CT_image_1"
          class="box-card"
          style="
            border-radius: 8px;
            width: 800px;
            height: 360px;
            margin-bottom: -30px;
          "
        >
          <div class="demo-image__preview1">
            <div
              v-loading="loading"
              element-loading-text="上传图片中"
              element-loading-spinner="el-icon-loading"
            >
              <el-image
                :src="url_1"
                class="image_1"
                :preview-src-list="srcList"
                style="border-radius: 3px 3px 0 0"
              >
                <div slot="error">
                  <div slot="placeholder" class="error">
                    <el-button
                      v-show="showbutton"
                      type="primary"
                      icon="el-icon-upload"
                      class="download_bt"
                      v-on:click="true_upload"
                    >
                      上传图像
                      <input
                        ref="upload"
                        style="display: none"
                        name="file"
                        type="file"
                        @change="update"
                      />
                    </el-button>
                  </div>
                </div>
              </el-image>
            </div>
            <div class="img_info_1" style="border-radius: 0 0 5px 5px">
              <span style="color: white; letter-spacing: 6px">原始图像</span>
            </div>
          </div>

          <div class="demo-image__preview2">
            <div
              v-loading="loading"
              element-loading-text="处理中,请耐心等待"
              element-loading-spinner="el-icon-loading"
            >
              <el-image
                :src="url_2"
                class="image_1"
                :preview-src-list="srcList1"
                style="border-radius: 3px 3px 0 0"
              >
                <div slot="error">
                  <div slot="placeholder" class="error">{{ wait_return }}</div>
                </div>
              </el-image>
            </div>
            <div class="img_info_1" style="border-radius: 0 0 5px 5px">
              <span style="color: white; letter-spacing: 4px">检测结果</span>
            </div>
          </div>

          
        </el-card>
      </div>

      
      <div float='left'>
                    <el-button
                      v-show="showbutton"
                      type="primary"
                      icon="el-icon-upload"
                      class="download_bt"
                      v-on:click="true_upload3"
                    >
                      上传视频
                      <input
                        ref="upload3"
                        style="display: none"
                        name="file"
                        type="file"
                        @change="update"
                      />
                    </el-button>

            <!-- <img src="require({{url_for('video_feed') }})"> -->

            <!-- <router-link :to="{name:'video_url'}"> 11  </router-link> -->
            <video
            :src="url_mp4"
            width="800" height="300" controls="controls" >
              <!-- <source src="url_mp4" type="video/mp4" /> -->
            </video>
            <template>
                <div class='demo'>
                    <video-player class="video-player vjs-custom-skin"
                                  ref="videoPlayer"
                                  :playsinline="true"
                                  :options="playerOptions">
                    </video-player>
                </div>
            </template>
        </div>

      </div>


      <div id="info_patient">
        <!-- 卡片放置表格 -->
        <el-card style="border-radius: 8px">
          <div slot="header" class="clearfix">
            <span>检测目标</span>
            <el-button
              style="margin-left: 35px"
              v-show="!showbutton"
              type="primary"
              icon="el-icon-upload"
              class="download_bt"
              v-on:click="true_upload2"
            >
              重新选择图像
              <input
                ref="upload2"
                style="display: none"
                name="file"
                type="file"
                @change="update"
              />
            </el-button>
          </div>
          <el-tabs v-model="activeName">
            <el-tab-pane label="检测到的目标" name="first">
              <!-- 表格存放特征值 -->
              <el-table
                :data="feature_list"
                height="2000"
                
                style="width: 1000px; text-align: center"
                v-loading="loading"
                element-loading-text="数据正在处理中，请耐心等待"
                element-loading-spinner="el-icon-loading"
                lazy
              >
                <el-table-column label="目标类别" width="250px">
                  <template slot-scope="scope">
                    <span>{{ scope.row[3] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="目标大小" width="250px">
                  <template slot-scope="scope">
                    <span>{{ scope.row[0] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="置信度" width="250px">
                  <template slot-scope="scope">
                    <span>{{ scope.row[1] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="crop" width="250px">
                  <template slot-scope="scope">
                    <el-image
                      :src="scope.row[2]"
                      class="image_1"
                      :preview-src-list="srcList1"
                      style="border-radius: 3px 3px 0 0"
                    >
                    </el-image>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </div>
      
    </div>
 
</template>

<script>
import 'vue-video-player/src/custom-theme.css'
import 'video.js/dist/video-js.css'

import axios from "axios";
export default {
  name: "Content",
  data() {
    return {
      server_url: "http://202.201.119.152:5003",
      activeName: "first",
      active: 0,
      centerDialogVisible: true,
      url_1: "",
      url_2: "",
      url_mp4:"",
      textarea: "",
      srcList: [],
      srcList1: [],
      feature_list: [],
      feature_list_1: [],
      feat_list: [],
      url: "",
      visible: false,
      wait_return: "等待上传",
      wait_upload: "等待上传",
      loading: false,
      table: false,
      isNav: false,
      showbutton: true,
      percentage: 0,
      fullscreenLoading: false,
      opacitys: {
        opacity: 0,
      },
      dialogTableVisible: false,
      url111:'http://127.0.0.1:5003/video/playphone-test2.mp4',
      playerOptions: {
                playbackRates: [0.5, 1.0, 1.5, 2.0], // 可选的播放速度
                autoplay: false,  // 如果为true,浏览器准备好时开始回放
                muted: false,     // 默认情况下将会消除任何音频。
                loop: false,      // 是否视频一结束就重新开始。
                preload: 'auto',  // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
                language: 'zh-CN',
                aspectRatio: '16:9',  // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
                fluid: true,  // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
                sources: [{
                    type: "video/mp4",  // 类型
                    src: ''        // url地址
                }],
                poster: '',  // 封面地址
                notSupportedMessage: '此视频暂无法播放，请稍后再试',  // 允许覆盖Video.js无法播放媒体源时显示的默认信息。
                controlBar: {
                    timeDivider: true,           // 当前时间和持续时间的分隔符
                    durationDisplay: true,       // 显示持续时间
                    remainingTimeDisplay: false, // 是否显示剩余时间功能
                    fullscreenToggle: true       // 是否显示全屏按钮
                }
            },
    };
  },
  created: function () {
    document.title = "YOLOv5目标检测WEB端";
  },
  methods: {
    true_upload() {
      this.$refs.upload.click();
    },
    true_upload2() {
      this.$refs.upload2.click();
    },
    true_upload3() {
      this.$refs.upload3.click();
    },
    next() {
      this.active++;
    },
    // 获得目标文件
    getObjectURL(file) {
      var url = null;
      if (window.createObjcectURL != undefined) {
        url = window.createOjcectURL(file);
      } else if (window.URL != undefined) {
        url = window.URL.createObjectURL(file);
      } else if (window.webkitURL != undefined) {
        url = window.webkitURL.createObjectURL(file);
      }
      return url;
    },
    // 上传文件
    update(e) {
      this.video_url=this.server_url+'/video/test2.mp4'
      this.percentage = 0;
      this.dialogTableVisible = true;
      this.url_mp4="";
      this.url_1 = "";
      this.url_2 = "";
      this.srcList = [];
      this.srcList1 = [];
      this.wait_return = "";
      this.wait_upload = "";
      this.feature_list = [];//值
      this.feat_list = [];//键
      this.fullscreenLoading = true;
      this.loading = true;
      this.showbutton = false;

      let file = e.target.files[0];
      this.url_1 = this.$options.methods.getObjectURL(file);
      let param = new FormData(); //创建form对象
      param.append("file", file, file.name); //通过append向form对象添加数据
      var timer = setInterval(() => {
        this.myFunc();
      }, 30);
      let config = {
        headers: { "Content-Type": "multipart/form-data" },
      }; //添加请求头
      axios
        .post(this.server_url + "/upload", param, config)
        .then((response) => {
          this.percentage = 100;
          clearInterval(timer);
          this.url_1 = response.data.image_url;
          this.srcList.push(this.url_1);
          // this.url_2 = response.data.draw_url;
          this.srcList1.push(this.url_2);
          this.fullscreenLoading = false;
          this.loading = false;
          alert(response.data)
          // this.url_mp4=response.data.url_mp4
          this.url_mp4=response.data.url_mp4;


          // this.url_mp4='http://127.0.0.1:5001/video/frame.mp4'
          //this.url_mp4='http://202.201.119.152:5003/video/test2.mp4'
          alert(this.url_mp4)
          //Object.keys 获取所有键
          //this.feat_list = Object.keys(response.data.image_info);
          //遍历键列表，取出键名'playphone-01'，通过键名查找每一个目标的值['150×117', 0.917] 放到值列表中
          //for (var i = 0; i < this.feat_list.length; i++) {
          //  response.data.image_info[this.feat_list[i]][3] = this.feat_list[i];//将键名加到值中方便传给表单
          //  this.feature_list.push(response.data.image_info[this.feat_list[i]]);
           
          //}
          //this.feature_list_1 = this.feature_list[0];
          this.dialogTableVisible = false;
          this.percentage = 0;
          this.notice1();
          this.playerOptions['sources'][0]['src']=this.url_mp4

        });
    },
    myFunc() {
      if (this.percentage + 33 < 99) {
        this.percentage = this.percentage + 33;
      } else {
        this.percentage = 99;
      }
    },
    drawChart() {},
    notice1() {
      this.$notify({
        title: "预测成功",
        message: "点击图片可以查看大图",
        duration: 0,
        type: "success",
      });
    },
  },
  mounted() {
    this.drawChart();
  },
};
</script>

<style>
.el-button {
  padding: 12px 20px !important;
}
#hello p {
  font-size: 15px !important;
  /*line-height: 25px;*/
}
.n1 .el-step__description {
  padding-right: 20%;
  font-size: 14px;
  line-height: 20px;
  /* font-weight: 400; */
}
</style>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
.dialog_info {
  margin: 20px auto;
}
.text {
  font-size: 14px;
}
.item {
  margin-bottom: 18px;
}
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}
.box-card {
  width: 680px;
  height: 200px;
  border-radius: 8px;
  margin-top: -20px;
}
.divider {
  width: 50%;
}
#CT {
  display: flex;
  height: 100%;
  width: 100%;
  flex-wrap: wrap;
  justify-content: center;
  margin: 0 auto;
  margin-right: 0px;
  max-width: 1800px;
}
#CT_image_1 {
  width: 90%;
  height: 40%;
  margin: 0px auto;
  padding: 0px auto;
  margin-right: 180px;
  margin-bottom: 0px;
  border-radius: 4px;
}
#CT_image {
  margin-bottom: 60px;
  margin-left: 30px;
  margin-top: 5px;
}
.image_1 {
  width: 275px;
  height: 260px;
  background: #ffffff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}
.img_info_1 {
  height: 30px;
  width: 275px;
  text-align: center;
  background-color: #21b3b9;
  line-height: 30px;
}
.demo-image__preview1 {
  width: 250px;
  height: 290px;
  margin: 20px 60px;
  float: left;
}
.demo-image__preview2 {
  width: 250px;
  height: 290px;
  margin: 20px 460px;
  /* background-color: green; */
}
.error {
  margin: 100px auto;
  width: 50%;
  padding: 10px;
  text-align: center;
}
.block-sidebar {
  position: fixed;
  display: none;
  left: 50%;
  margin-left: 600px;
  top: 350px;
  width: 60px;
  z-index: 99;
}
.block-sidebar .block-sidebar-item {
  font-size: 50px;
  color: lightblue;
  text-align: center;
  line-height: 50px;
  margin-bottom: 20px;
  cursor: pointer;
  display: block;
}
div {
  display: block;
}
.block-sidebar .block-sidebar-item:hover {
  color: #187aab;
}
.download_bt {
  padding: 10px 16px !important;
}
#upfile {
  width: 104px;
  height: 45px;
  background-color: #187aab;
  color: #fff;
  text-align: center;
  line-height: 45px;
  border-radius: 3px;
  box-shadow: 0 0 2px 0 rgba(0, 0, 0, 0.1), 0 2px 2px 0 rgba(0, 0, 0, 0.2);
  color: #fff;
  font-family: "Source Sans Pro", Verdana, sans-serif;
  font-size: 0.875rem;
}
.file {
  width: 200px;
  height: 130px;
  position: absolute;
  left: -20px;
  top: 0;
  z-index: 1;
  -moz-opacity: 0;
  -ms-opacity: 0;
  -webkit-opacity: 0;
  opacity: 0; /*css属性&mdash;&mdash;opcity不透明度，取值0-1*/
  filter: alpha(opacity=0);
  cursor: pointer;
}
#upload {
  position: relative;
  margin: 0px 0px;
}
#Content {
  width: 85%;
  height: 800px;
  background-color: #ffffff;
  margin: 15px auto;
  display: flex;
  min-width: 1200px;
}
.divider {
  background-color: #eaeaea !important;
  height: 2px !important;
  width: 100%;
  margin-bottom: 50px;
}
.divider_1 {
  background-color: #ffffff;
  height: 2px !important;
  width: 100%;
  margin-bottom: 20px;
  margin: 20px auto;
}
.steps {
  font-family: "lucida grande", "lucida sans unicode", lucida, helvetica,
    "Hiragino Sans GB", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
  color: #21b3b9;
  text-align: center;
  margin: 15px auto;
  font-size: 20px;
  font-weight: bold;
  text-align: center;
}
.step_1 {
  /*color: #303133 !important;*/
  margin: 20px 26px;
}
#info_patient {
  margin-top: 10px;
  margin-right: 160px;
}
</style>
