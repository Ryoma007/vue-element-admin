<template>
  <div
    v-loading="loading"
  >
    <el-card class="container search-form">
      <ele-form
        ref="form-container"
        v-model="formData"
        :form-desc="formDesc"
        v-bind="searchFormOpt"
        :is-responsive="false"
        :request-fn="refreshTableData"
      />
    </el-card>
    <el-card class="container">
      <div class="table container">
        <el-table
          ref="table"
          :max-height="tableHeight"
          v-bind="tableOpt"
          v-on="tableEvent"
        >
          <slot name="column" />
        </el-table>
      </div>
      <div ref="page" class="page container">
        <el-pagination
          :current-page="formData.page"
          v-bind="pageOpt"
          :page-size="formData.rows"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
  </div>
</template>

<script>
import { throttle } from 'lodash'
export default {
  name: 'DataTableContainer',
  props: {
    pageOpt: {
      type: Object,
      default: function() {
        return {
          pageSizes: [10, 20, 30, 40],
          layout: 'total, sizes, prev, pager, next, jumper'
        }
      }
    },
    total: {
      type: Number,
      required: true
    },
    refreshData: {
      type: Function,
      required: true
    },
    tableOpt: {
      type: Object,
      required: true,
      validator: value => {
        const { data } = value
        if (!data) {
          console.error('缺少data')
          return false
        }
        return true
      }
    },
    tableEvent: {
      type: Object,
      default: null
    },
    searchFormOpt: {
      type: Object,
      default: function() {
        return {
          isShowSubmitBtn: true,
          isShowBackBtn: false,
          isShowResetBtn: true,
          isShowCancelBtn: false,
          submitBtnText: '查询',
          formBtns: [
            {
              text: '导出',
              type: 'primary',
              click: () => {
                // this.$message('todo')
                this.loading = true
                setTimeout(() => {
                  this.loading = false
                }, 3000)
              }
            }
          ]
        }
      }
    },
    formDesc: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      loading: false,
      refreshTableHeightThr: null,
      tableHeight: null,
      formData: {
        page: 1,
        rows: 10
      }
    }
  },
  mounted() {
    this.refreshTableHeightThr = throttle(this.refreshTableHeight, 500)
    this.$nextTick(() => {
      this.refreshTableHeightThr()
    })
    window.onresize = this.refreshTableHeightThr
  },
  methods: {
    refreshTableHeight() {
      const el = this.$refs.table.$el
      const { top: offsetTop } = this.getOffset(el)
      const outerBottomHeight = this.getAllOuterBottomHeight(el)
      const pageHeight = this.getHeight(this.$refs.page)
      const windowHeight = window.document.documentElement.clientHeight
      this.tableHeight = windowHeight - offsetTop - pageHeight - outerBottomHeight
    },
    getHeight(el) {
      return el.getBoundingClientRect().height + this.getOuterHeight(el)
    },
    getAllOuterBottomHeight(el, val = 0) {
      val += parseFloat(this.getCss(el, 'padding-bottom') || 0)
      val += parseFloat(this.getCss(el, 'margin-bottom') || 0)
      if (el.parentNode) {
        return this.getAllOuterBottomHeight(el.parentNode, val)
      }
      return val
    },
    getOuterHeight(el) {
      let val = 0
      val += parseFloat(this.getCss(el, 'padding-top') || 0)
      val += parseFloat(this.getCss(el, 'padding-bottom') || 0)
      val += parseFloat(this.getCss(el, 'margin-top') || 0)
      val += parseFloat(this.getCss(el, 'margin-bottom') || 0)
      return val
    },
    getCss(el, name) {
      try {
        if (!el || !el.ownerDocument) { return null }
        // Native
        // 注意：此处为了解决当 style 值为 auto 时，返回 auto 的问题
        const win = el.ownerDocument.defaultView
        // null 的意思是不返回伪类元素
        return win.getComputedStyle(el, null)[name]
      } catch (e) {
        debugger
        console.error(e)
      }
    },
    getOffset(el) {
      const box = el.getBoundingClientRect()
      return {
        top: box.top + window.pageYOffset - document.documentElement.clientTop,
        left: box.left + window.pageXOffset - document.documentElement.clientLeft
      }
    },
    handleSizeChange(size) {
      this.formData.rows = size
      this.refreshTableData()
    },
    handleCurrentChange(page) {
      this.formData.page = page
      this.refreshTableData()
    },
    async refreshTableData() {
      try {
        this.loading = true
        await this.refreshData(this.formData)
      } finally {
        this.loading = false
      }
    }
  }
}

</script>

<style scoped>
  .container{
    margin: 5px 0;
  }
</style>
