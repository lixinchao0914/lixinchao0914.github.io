<template>
  <div class="page-box">
    <div>
      <div class="table-tools">
        <el-button
          type="primary"
          @click="clickImport"
        >导入</el-button>
        <el-button
          type="primary"
          :disabled="downloadLoading"
          :loading="downloadLoading"
          @click="handleDownload"
        >下载</el-button>
      </div>

      <!--      height="600"-->
      <el-table
        :data="list"
        style="width: 100%"
        stripe
        border
      >
        <el-table-column
          type="index"
          width="100"
          label="序号"
          fixed
          :index="(index)=>index+1"
        />
        <el-table-column
          v-for="info in tableColKeys"
          :key="info.label"
          :label="info.label"
          :align="info.align ||'center'"
        >
          <template v-if="info.children">
            <el-table-column
              v-for="item in info.children"
              :key="item.prop"
              :column-key="item.prop"
              header-align="center"
              :align="item.align"
              :prop="item.prop"
              :label="item.label"
              :width="item.width"
            >
              <template slot-scope="scope">
                <el-tooltip
                  :disabled="!item.openTooltip"
                  class="item"
                  effect="dark"
                  :content="getValue(scope.row, item.prop)"
                  placement="top-start"
                >
                  <p class="u-line-2">{{ getValue(scope.row, item.prop) }}</p>
                </el-tooltip>

              </template>
            </el-table-column>
          </template>

        </el-table-column>
      </el-table>
    </div>
    <el-dialog
      title="导入数据"
      :visible.sync="dialogVisible"
      width="500px"
      append-to-body
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      :show-close="false"
      :before-close="handleClose"
    >
      <div class="textarea-list">
        <div
          v-for="(item, index) in importList"
          :key="index"
          class="textarea-item"
        >
          <div class="index-name">{{ index + 1 }}：</div>
          <el-input
            v-model="item.textarea"
            type="textarea"
            placeholder="输入(full)Response复制的数据"
            :autosize="{ minRows: 2, maxRows: 2}"
            clearable
          />
        </div>

      </div>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="handleClose">取 消</el-button>
        <el-button
          type="primary"
          @click="confirmImport"
        >导 入</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'HomePage',
  data() {
    return {
      downloadLoading: false,
      dialogVisible: false,
      importList: [
        {
          textarea: ''
        },
        {
          textarea: ''
        },
        {
          textarea: ''
        },
        {
          textarea: ''
        },
        {
          textarea: ''
        }
      ],
      cloneImportList: [],
      list: [],
      tableColKeys: [
        {
          label: '视频信息',
          align: 'center',
          children: [
            {
              label: '视频id',
              prop: 'id',
              align: 'left',
              width: '180'
            },
            {
              label: '视频播放地址',
              prop: 'videoPlayUrl',
              align: 'left',
              width: '180'
            },
            {
              label: '视频描述',
              prop: 'desc',
              align: 'left',
              width: '220',
              openTooltip: true
            },
            {
              label: '视频播放数',
              prop: 'stats.playCount',
              align: 'right',
              width: '120'
            },
            {
              label: '视频收藏数',
              prop: 'stats.collectCount',
              align: 'right',
              width: '120'
            },
            {
              label: '视频评论数',
              prop: 'stats.commentCount',
              align: 'right',
              width: '120'
            }
          ]
        }, {
          label: '博主信息',
          align: 'center',
          children: [
            {
              label: '博主昵称',
              prop: 'author.nickname',
              align: 'left',
              width: '180'
            },
            {
              label: '博主主页',
              prop: 'authorUrl',
              align: 'left',
              width: '180'
            },
            {
              label: '博主粉丝数',
              prop: 'authorStats.followerCount',
              align: 'right',
              width: '120'
            },
            {
              label: '博主总点赞数',
              prop: 'authorStats.heartCount',
              align: 'right',
              width: '120'
            },
            {
              label: '博主视频数',
              prop: 'authorStats.videoCount',
              align: 'right',
              width: '120'
            },
            {
              label: '博主签名',
              prop: 'author.signature',
              align: 'left',
              width: '180',
              openTooltip: true
            },
            {
              label: '邮箱',
              prop: 'email',
              align: 'left',
              width: '150'
            }
          ]
        }
      ]
    }
  },
  created() {
    this.cloneImportList = JSON.parse(JSON.stringify(this.importList))
  },
  methods: {
    handleClose() {
      this.dialogVisible = false
    },
    // 导入
    clickImport() {
      this.importList = JSON.parse(JSON.stringify(this.cloneImportList))
      this.dialogVisible = true
    },
    // 确认导入
    confirmImport() {
      try {
        let list = []
        this.importList.forEach(item => {
          const str = item.textarea.trim()
          if (str) {
            list = list.concat(JSON.parse(str).item_list)
          }
        })
        this.list = this.formListData(list)
        this.handleClose()
      } catch (e) {
        this.$message.error('数据错误')
      }
    },
    // 获取文本中的邮箱地址
    getMailByText(text) {
      const emailRegex = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g
      const mailArr = text.match(emailRegex) || []
      return mailArr.join('，')
    },
    // 格式化列表
    formListData(list) {
      return list.map(item => {
        return {
          ...item,
          email: this.getMailByText(item.author.signature),
          authorUrl: `https://www.tiktok.com/@${item.author.uniqueId}`,
          videoPlayUrl: `https://www.tiktok.com/@${item.author.uniqueId}/video/${item.id}`
        }
      })
    },
    // 下载
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = []
        this.tableColKeys.forEach(item => {
          if (item.children) {
            item.children.forEach(item2 => {
              tHeader.push(item2.label)
            })
          } else {
            tHeader.push(item.label)
          }
        })
        const filterVal = []
        this.tableColKeys.forEach(item => {
          if (item.children) {
            item.children.forEach(item2 => {
              filterVal.push(item2.prop)
            })
          } else {
            filterVal.push(item.prop)
          }
        })
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    // 格式化excel表格
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return 'timestamp'
        } else {
          return this.getValue(v, j)
        }
      }))
    },
    // 获取对象值 多级
    getValue(obj, key) {
      let curVal = obj
      const keys = key.split('.')
      keys.forEach(key => {
        curVal = curVal[key]
      })
      return curVal
    }
  }
}
</script>

<style scoped lang="scss">
.page-box{
  width: 100vw;
  height: 100vh;
}
.table-tools{
  padding: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>
<style lang="scss">
.textarea-list{
  overflow-y: auto;
  //max-height: 500px;
  .textarea-item + .textarea-item{
    margin-top: 10px;
  }
  .textarea-item{

  }
}
</style>
