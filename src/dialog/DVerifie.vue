<template>
  <el-dialog :title="apiInfo.publishedUserName+' 提交申请'" :size="size" v-model="Visible" @close="close">
    <div class="content">
      <div class="info">
        <div class="desc">
          申请说明：{{apiInfo.publishedDescription||'无'}}
        </div>
        <div class="time">{{apiInfo.update_time|datetime}}</div>
      </div>
      <hr>
      <div class="doc">
        <doc-viewer :apiInfo="apiInfoJson"></doc-viewer>
      </div>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="refuse">拒绝</el-button>
      <el-button type="primary" @click="pass">通过</el-button>
    </div>
  </el-dialog>
</template>
<style lang="styl" rel="stylesheet/stylus" scoped type="text/css">
  .content
    .info
      position relative
      .desc
        padding 10px
      .time
        position absolute
        bottom -15px
        right 0
    .doc
      max-height 500px
      overflow: auto
</style>
<script type="text/ecmascript-6">
  import BaseDialog from 'src/extend/BaseDialog'
  import jsYaml from 'js-yaml'
  import Server from 'src/extend/Server'
  import DocViewer from 'src/components/DocViewer'

  export default {
    mixins: [ BaseDialog ],
    name: 'DVerifie',
    data: function () {
      return {
        apiInfoJson: {},
        apiInfo: {}
      }
    },
    components: { DocViewer },
    computed: {},
    mounted: function () {
      this.loadApiInfo()
    },
    methods: {
      loadApiInfo: function () {
        Server({
          url: 'api/getInterfaceInfo',
          params: {
            apiId: this.id,
            type: 2
          },
          method: 'get'
        }).then((response) => {
          var data = response.data.data
          this.apiInfo = data
          this.apiInfoJson = jsYaml.safeLoad(this.apiInfo.content) || {}
        }).catch((e) => {

        })
      },
      audit: function (value, status) {
        Server({
          url: 'api/audit',
          data: {
            apiId: this.apiInfo.id,
            description: value,
            status: status
          },
          method: 'post'
        }).then((response) => {
          this.close()
          this.$message('修改成功')
          this.callback(status)
        }).catch(() => {
          this.close()
        })
      },
      refuse: function () {
        this.$prompt('输入备注', '拒绝发布', {
          inputPattern: /.{0,30}/,
          inputErrorMessage: '备注0到30个字'
        }).then(({ value }) => {
          this.audit(value, 2)
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          })
        })
      },
      pass: function () {
        this.$prompt('输入备注', '通过发布', {
          inputPattern: /.{0,30}/,
          inputErrorMessage: '备注0到30个字'
        }).then(({ value }) => {
          this.audit(value, 1)
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          })
        })
      },
      onSubmit () {
        console.log('submit!')
      }
    }
  }
</script>

