<template>
  <div style="padding: 20px 50px">
    <h1 class="title">关于proxyee-down</h1>
    <el-row>
      <el-col :span="3">
        <span>项目主页</span>
      </el-col>
      <el-col :span="16">
        <span>
          <a target="_blank"
             href="https://github.com/monkeyWie/proxyee-down">GitHub@proxyee-down</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="3">
        <span>使用教程</span>
      </el-col>
      <el-col :span="16">
        <span>
          <a target="_blank"
             href="https://github.com/monkeyWie/proxyee-down/blob/master/README.md">@官方编写</a>
        </span>
        <span style="padding-left: 20px">
          <a target="_blank"
             href="https://www.zhanghuanglong.com/detail/another-download-of-magic-proxyee-down">@°只为大大编写</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="3">
        <span>软件下载</span>
      </el-col>
      <el-col :span="16">
        <span>
          <a target="_blank"
             href="https://github.com/monkeyWie/proxyee-down/releases">GitHub@proxyee-down/releases</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="3">
        <span>BUG提交</span>
      </el-col>
      <el-col :span="16">
        <span>
          <a target="_blank" href="https://github.com/monkeyWie/proxyee-down/issues">GitHub@proxyee-down/issue</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="3">
        <span>QQ群</span>
      </el-col>
      <el-col :span="16">
        <span>11352304</span>
      </el-col>
    </el-row>
    <h1 class="title">软件升级</h1>
    <el-row>
      <el-col :span="4">
        <span>当前版本：{{version}}</span>
      </el-col>
      <el-col :span="6">
        <el-button @click="checkUpdate" type="primary" :loading="checkLoading">检测更新</el-button>
        <el-tooltip class="item" content="若更新进度无响应可重新检查更新" placement="right">
          <i class="el-icon-question" style="position: relative;top:-15px;"></i>
        </el-tooltip>
      </el-col>
      <el-dialog
        title="更新提示"
        :visible.sync="dialogVisible"
        width="30%">
        <h2>检测到新版本</h2>
        <h3>版本号：{{updateInfo.version}}</h3>
        <h3>更新内容：</h3>
        <div v-html="updateInfo.desc"></div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">关闭</el-button>
          <el-button type="primary" @click="doUpdate()" :loading="updateLoading">更新</el-button>
        </span>
      </el-dialog>
    </el-row>
    <el-row v-if="!!updateTask">
      <el-col :span="8">
        <el-progress :percentage="progress(updateTask)" :status="status(updateTask)"></el-progress>
      </el-col>
    </el-row>
    <h1 class="title">感谢以下开源项目</h1>
    <el-row>
      <el-col :span="16">
        <span>
          <a target="_blank" href="http://spring.io/">spring</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="16">
        <span>
          <a target="_blank" href="http://netty.io/">netty</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="16">
        <span>
          <a target="_blank" href="https://cn.vuejs.org/">vue.js</a>
        </span>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="16">
        <span>
          <a target="_blank" href="http://element.eleme.io">Element UI</a>
        </span>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        version: '',
        checkLoading: false,
        updateLoading: false,
        dialogVisible: false,
        updateInfo: {},
        intervalId: null,
        updateTask: null
      }
    },
    methods: {
      checkUpdate() {
        this.checkLoading = true;
        this.$http.get('api/checkUpdate')
        .then((response) => {
          let result = response.data;
          if (result.status == 200) {
            this.updateInfo = result.data;
            this.updateInfo.version = this.updateInfo.version.toFixed(2);
            this.dialogVisible = true;
          } else {
            this.$message({showClose: true, message: result.msg});
          }
          this.checkLoading = false;
        });
      },
      doUpdate() {
        this.updateLoading = true;
        this.$http.get('api/doUpdate')
        .then((response) => {
          let result = response.data;
          if (result.status == 200) {
          } else {
            this.$message({showClose: true, message: result.msg});
          }
          this.dialogVisible = false;
          this.updateLoading = false;
        });
      },
      progress(task) {
        let fileDownSize = task.downSize;
        let fileTotalSize = task.totalSize;
        if (fileDownSize > 0 && fileTotalSize > 0) {
          return Math.floor(fileDownSize * 100 / fileTotalSize);
        }
        return 0;
      },
      status(task) {
        switch (task.status) {
          case 7:
            return 'success';
          default:
            return null;
        }
      },
    },
    created() {
      this.$http.get('api/getVersion')
      .then((response) => {
        let result = response.data;
        if (result.status == 200) {
          this.version = result.data.toFixed(2);
        } else {
          this.$message({showClose: true, message: result.msg});
        }
      });
      this.intervalId = setInterval(() => {

        this.$http.get('api/getUpdateProgress')
        .then((response) => {
          let result = response.data
          if (result.status == 200 && result.data) {
            this.updateTask = result.data;
            if (this.updateTask.status == 7) {
              clearInterval(this.intervalId);
              this.$confirm('更新完成是否重启程序？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
              }).then(() => {
                this.$http.get('api/restart')
                .then(() => {
                  window.location.href = "about:blank";
                }).catch(() => {
                  window.location.href = "about:blank";
                });
              }).catch(() => {
              });
            }
          }
        });
      }, 1000);
    },
    destroyed() {
      if (this.intervalId) {
        clearInterval(this.intervalId);
      }
    }
  }
</script>

<style scoped>
  .el-row {
    padding-top: 10px;
  }

  .title {
    font-size: 28px;
  }

  span {
    font-size: 18px;
  }
</style>
