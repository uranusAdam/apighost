<template>
  <div class="page">
    <div class="layout-nav">
      <div class="container-fluid">
        <div class="leftcontrols">
          <span class="members">
            <span class="member">
              <img src="http://ok0m4xukx.bkt.clouddn.com/b8ad7b37c8a20c71aa346271a9aca38d.jpg" alt="">
            </span>
            <el-popover
                ref="searchmember"
                width="400"
                trigger="click">
                <div>这里需要搜索用户组件</div>
            </el-popover>
            <span v-popover:searchmember class="member add">
              <i class="ifont icon-add"></i>
            </span>
          </span>
          <span class="tag">
            <i class="ifont icon-tag"></i>
            <el-tag
                v-for="(tag,key) in dynamicTags"
                :closable="true"
                :close-transition="false"
                @close="handleClose(key,tag)"
            >
              {{tag.name}}
            </el-tag>
            <el-input
                class="input-new-tag"
                v-if="inputVisible"
                v-model="inputValue"
                ref="saveTagInput"
                size="mini"
                :autofocus="true"
                :maxlength="10"
                :minlength="1"
                @keyup.enter.native="handleInputConfirm"
                @blur="handleInputConfirm"
            >
            </el-input>
            <el-button v-if="inputBtnVisible" class="button-new-tag" size="small"
                       @click="inputVisible = true,inputBtnVisible = false">+新标签</el-button>
          </span>
        </div>
        <div class="controls">

          <el-button v-if="apiInfo.status==1" @click="applyPublish" type="warning" size="small">
            <el-tooltip content="发布后会通知管理者，管理者统一后该api的修改能同步导项目中">
              申请发布 <i class="ifont icon-menu"></i>
            </el-tooltip>
          </el-button>
          <el-dropdown v-if="apiInfo.status==1" split-button size="small" type="primary" @command="handleCommand"
                       @click="upDateApi">
            保存
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="publish">保存并发布</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <el-popover
              ref="setting"
              width="200"
              trigger="click">
            <div class="dropdown-menu-nav">
              <ul>
                <li @click="copyNew">
                  <a aria-label="Profile Settings">拷贝并新建</a>
                </li>
                <li @click="editMock">
                  <a aria-label="Profile Settings">MOCK数据</a>
                </li>
                <li class="divider"></li>
                <li @click="delectApi">
                  <a class="sign-out-link" aria-label="Sign out" rel="nofollow" data-method="delete">删除接口</a>
                </li>
              </ul>
            </div>
          </el-popover>
          <el-button v-popover:setting type="primary" size="small">
            <i class="el-icon-setting"></i>
          </el-button>
        </div>
      </div>
    </div>
    <div class="content-wrapper page-with-layout-nav" :style="{height: (appInfo.size.height-50-58) + 'px'}">
      <div style="width: 100%">
        <el-alert v-if="apiInfo.status==2"
                  title="注意该接口处于审核中状态。修改不会被保存。"
                  type="warning">
        </el-alert>
        <div class="content">
          <div class="editor">
            <yaml-editer :contents="apiInfo.content" :on-change="editorChange"></yaml-editer>
          </div>

          <div class="preview">
            <doc-viewer :apiInfo="apiInfoJson"></doc-viewer>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="styl" rel="stylesheet/stylus" type="text/css">
  .page
    overflow hidden
    display box
    -webkit-box-orient: vertical;
    .el-dropdown
      .el-button-group
        display inline-block
    .leftcontrols
      display inline-block
      float: left;
      padding: 10px 0;
      .members
        display inline-block
        height 25px
        .member
          background-color #ddd
          width: 25px
          height 25px
          border-radius 50%
          overflow hidden
          display inline-block
          img
            width 100%
        .add
          background-color #fafafa
          color #1a98ea
          border-radius 0%
          .icon-add
            font-size 25px
            line-height 29px
      .tag
        margin-left 40px
        position: relative;
        bottom: 7px;
        .icon-tag
          font-size 20px
        .input-new-tag
          display inline-block
          width 100px
        .el-tag
          margin-left 3px
    .layout-nav
      .controls-left
        float left
    .content-wrapper
      display flex
      flex 1
      width 100%
      .content
        display flex
        height 100%
        width 100%
        .editor
          height 100%
          flex 1
          border-right 1px solid #ddd
          .econ
            height 100%
            width 100%
        .preview
          padding 5px
          height 100%
          flex 1
          overflow auto
</style>

<script type="text/ecmascript-6">
  import BasePage from 'src/extend/BasePage'
  import Server from 'src/extend/Server'
  import CodeViewer from 'src/components/CodeViewer'
  import DocViewer from 'src/components/DocViewer'
  import YamlEditer from 'src/components/YamlEditer'
  import jsYaml from 'js-yaml'
  import {mapState} from 'vuex'
  export default{
    mixins: [ BasePage ],
    components: { CodeViewer, DocViewer, YamlEditer },
    name: 'api_new',
    data () {
      return {
        dynamicTags: [],
        inputVisible: false,
        inputBtnVisible: true,
        inputValue: '',
        apiInfoJson: {},
        apiInfo: {
          id: '',
          content: `path: /demo
name: 接口demo${Math.random()}
method: get
description: 获取用fdsafdsa户姓名
parameters:
  body:
    name!string: 内容
    isCat!boolean:
    body:
      age!number: 1
  path:
  query:
responses:
  200:
    a: 1`,
          description: '描述',
          name: '接口demo',
          method: 'get',
          path: 'demo',
          status: 1,
          projectId: '',
          tags: []
        }
      }
    },
    computed: mapState({
      appInfo: state => state.app
    }),
    mounted: function () {
      var id = this.$route.query.id
      this.addApi(id)
    },
    methods: {
      /**
       * 添加api接口
       */
      addApi: function (id) {
        if (id) {
          this.apiInfo.id = id
          Server({
            url: 'api/getInterfaceInfo',
            params: {
              apiId: id,
              type: 2
            },
            method: 'get'
          }).then((response) => {
            var data = response.data.data
            this.apiInfo = data
            this.dynamicTags = data.tags || []
          }).catch((e) => {

          })
        } else {
          Server({
            url: 'api/add',
            data: {
              content: this.apiInfo.content,
              description: this.apiInfo.description,
              method: this.apiInfo.method,
              name: this.apiInfo.name,
              path: this.apiInfo.path,
              projectId: this.apiInfo.projectId || this.$route.query.pid
            },
            method: 'post'
          }).then((response) => {
            var data = response.data.data
            this.apiInfo.id = data.id
            this.apiInfo.status = 1
          }).catch((e) => {
          })
        }
      },
      upDateApi: function () {
        Server({
          url: 'api/update',
          data: {
            apiId: this.apiInfo.id,
            content: this.apiInfo.content,
            remark: this.apiInfo.description,
            name: this.apiInfo.name,
            method: this.apiInfo.method,
            publishStatus: '1',
            path: this.apiInfo.path
          },
          method: 'put'
        }).then((response) => {
          this.$message('修改成功')
        }).catch((e) => {

        })
      },
      /* =============tag=========== */
      /**
       * 删除标签
       */
      handleClose (key, tag) {
        Server({
          url: 'api/deleteApiTag',
          data: {
            apiId: this.apiInfo.id,
            tagId: tag.id
          },
          method: 'delete'
        }).then((response) => {
          this.dynamicTags.splice(this.dynamicTags.indexOf(key), 1)
          if (this.dynamicTags.length > 2) {
            this.inputBtnVisible = false
          } else {
            this.inputBtnVisible = true
          }
        }).catch((e) => {
          this.$message('删除失败')
        })
      },
      /**
       * 添加标签
       */
      handleInputConfirm () {
        let inputValue = this.inputValue
        if (!inputValue) {
          return
        }
        Server({
          url: 'api/addApiTag',
          data: {
            apiId: this.apiInfo.id,
            tagName: inputValue
          },
          method: 'post'
        }).then((response) => {
          var data = response.data.data
          this.dynamicTags.push({ id: data.tagId, name: inputValue })
          this.inputVisible = false
          this.inputValue = ''
          if (this.dynamicTags.length > 2) {
            this.inputBtnVisible = false
          } else {
            this.inputBtnVisible = true
          }
        }).catch((e) => {
          this.$message('标签添加失败')
        })
      },
      /* =============事件=========== */

      /**
       * 编辑内容改变调用事件
       */
      editorChange: function (data) {
        this.apiInfo.content = data
        this.apiInfoJson = jsYaml.safeLoad(this.apiInfo.content) || {}
        this.apiInfoJson.id = this.apiInfo.id
        this.apiInfoJson.type = this.apiInfo.type
        this.apiInfoJson.mockResponse = this.apiInfo.mockResponse
        this.apiInfoJson.mockRequest = this.apiInfo.mockRequest
        this.apiInfo.path = this.apiInfoJson.path
        this.apiInfo.method = this.apiInfoJson.method
        this.apiInfo.name = this.apiInfoJson.name
      },
      /**
       * 删除api
       */
      delectApi: function () {
        this.$confirm(`确认删除${this.apiInfo.path}接口`, '提示', {
          type: 'warning'
        }).then(() => {
          // todo 服务端登陆接口访问
          Server({
            url: 'api/delete',
            method: 'delete',
            data: {
              apiId: this.apiInfo.id
            }
          }).then((response) => {
            this.$router.push({ path: 'dashboard_projects' })
          }).catch(() => {
            this.$message('删除失败')
          })
        }).catch(() => {
          this.$message('已取消')
        })
      },
      /**
       * 申请发布项目
       */
      applyPublish: function () {
        this.$prompt('输入备注', '申请发布', {
          inputPattern: /.{0,30}/,
          inputErrorMessage: '备注0到30个字'
        }).then(({ value }) => {
          Server({
            url: 'api/requestRelease',
            method: 'post',
            data: {
              apiId: this.apiInfo.id,
              description: value
            }
          }).then((response) => {
            this.apiInfo.status = 2
            this.$message('申请成功')
          }).catch(() => {
            this.$message('发布失败')
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          })
        })
      },

      /**
       * 发布项目
       */
      publish: function () {
        this.$prompt('输入备注', '发布', {
          inputPattern: /.{0,30}/,
          inputErrorMessage: '备注0到30个字'
        }).then(({ value }) => {
          Server({
            url: 'api/update',
            data: {
              apiId: this.apiInfo.id,
              content: this.apiInfo.content,
              remark: value,
              name: this.apiInfo.name,
              method: this.apiInfo.method,
              publishStatus: '3',
              path: this.apiInfo.path
            },
            method: 'put'
          }).then((response) => {
            this.$message('修改成功')
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          })
        })
      },

      /**
       * 拷贝并创建
       */
      copyNew: function () {
        this.addApi()
      },
      /**
       * 编辑api Mock文档
       */
      editMock: function () {
        this.openDialog({
          name: 'DMock',
          data: {
            title: '修改接口mock规则',
            apiId: this.apiInfo.id
          },
          methods: {}
        })
      },
      handleCommand (command) {
        if (command == 'publish') {
          if (this.apiInfo.role < 3) {
            this.publish()
          } else {
            this.$message('需要管理员权限才能保存并发布。请申请发布等待管理员审核。')
          }
        }
      }
    }
  }
</script>
