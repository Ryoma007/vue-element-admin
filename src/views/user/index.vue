<template>
  <div v-loading="loading">
    <data-table-container
      ref="datatable"
      :form-desc="formDesc"
      :table-opt="tableOpt"
      :table-event="tableEvent"
      :refresh-data="list"
      :total="total"
    >
      <template v-slot:column>
        <el-table-column
          type="selection"
          width="55"
        />
        <el-table-column
          type="index"
          width="50"
          label="序号"
        />
        <el-table-column
          property="id"
          label="ID"
          width="80"
        />
        <el-table-column
          property="name"
          label="名称"
          width="80"
        />
        <el-table-column
          property="code"
          label="用户代码"
          width="80"
        />
        <el-table-column
          property="status"
          label="状态"
          width="80"
          align="center"
        >
          <template slot-scope="scope">
            {{ scope.row.status===0?'禁用':'启用' }}
          </template>
        </el-table-column>
        <el-table-column
          property="title"
          label="头衔"
          width="80"
          align="center"
        />
        <el-table-column
          property="mobile"
          label="手机"
          width="140"
          align="center"
        />
        <el-table-column
          property="createTime"
          width="160"
          label="创建时间"
        />
        <el-table-column
          property="deptName"
          label="所属部门"
          width="80"
          align="center"
        />
        <el-table-column
          property="userGroupName"
          label="所属用户组"
          width="100"
          align="center"
        />
        <el-table-column
          property="roleName"
          label="所拥有角色"
          width="100"
          align="center"
        />
        <el-table-column
          label="操作"
          min-width="400"
        >
          <template slot-scope="scope">
            <el-button type="primary">编辑</el-button>
            <el-button type="primary">改密</el-button>
            <el-button type="warning">{{ scope.row.status===0?'启用':'禁用' }}</el-button>
            <el-button type="danger">删除</el-button>
            <el-button type="primary">设置角色</el-button>
            <el-button type="primary">日志</el-button>
          </template>
        </el-table-column>
      </template>
    </data-table-container>
  </div>
</template>

<script>
import DataTableContainer from '@/components/DataTableContainer'
import { list } from '@/api/user'

export default {
  name: 'UserIndex',
  components: { DataTableContainer },
  data() {
    return {
      loading: false,
      total: 0,
      tableOpt: {
        data: []
      },
      tableEvent: {
        select(selection, row) {
          console.log({ selection, row })
        }
      },
      'formDesc': {
        'name': {
          'label': '用户名称',
          'type': 'input',
          'layout': 6
        },
        'code': {
          'label': '用户代码',
          'type': 'input',
          'layout': 6
        },
        'deptName': {
          'options': [
            {
              'text': '研发部',
              'value': '研发部'
            },
            {
              'text': '综管部',
              'value': '综管部'
            },
            {
              'text': 'IT部',
              'value': 'IT部'
            },
            {
              'text': '质量部',
              'value': 4
            }
          ],
          'label': '部门',
          'type': 'select',
          'attrs': {
            'multiple': true,
            'clearable': true,
            'collapseTags': true
          },
          'layout': 6
        },
        'userGroupName': {
          'options': [
            {
              'text': '开发组',
              'value': '开发组'
            },
            {
              'text': '测试组',
              'value': '测试组'
            },
            {
              'text': '运维组',
              'value': '运维组'
            }
          ],
          'label': '用户组',
          'type': 'select',
          'attrs': {
            'multiple': true,
            'clearable': true,
            'collapseTags': true
          },
          'layout': 6
        },
        'status': {
          'options': [
            {
              'text': '启用',
              'value': 1
            },
            {
              'text': '禁用',
              'value': 0
            }
          ],
          'label': '状态',
          'type': 'select',
          'attrs': {
            'clearable': true
          },
          'layout': 6
        },
        'roleName': {
          'options': [
            {
              'text': '开发',
              'value': '开发'
            },
            {
              'text': '测试',
              'value': '测试'
            },
            {
              'text': '运维',
              'value': '运维'
            }
          ],
          'label': '角色',
          'type': 'select',
          'attrs': {
            'multiple': true,
            'clearable': true,
            'collapseTags': true
          },
          'layout': 6
        },
        'mobile': {
          'label': '手机',
          'type': 'input',
          'layout': 6
        }
      }
    }
  },
  mounted() {
    this.list()
    console.log(this.$refs.datatable)
  },
  methods: {
    async list(query) {
      try {
        this.loading = true
        const resp = await list(query)
        if (resp.data.data) {
          this.tableOpt.data = resp.data.data
          this.total = resp.data.total
        } else {
          console.error(resp)
          this.$message.error('获取数据失败')
        }
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>

</style>
