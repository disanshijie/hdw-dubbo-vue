<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.departmentName" placeholder="部门名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('enterprise/enterpriseDepartment/save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="dataList"
      border
      style="width: 100%;">
      <el-table-column
        prop="enterpriseName"
        header-align="center"
        align="center"
        label="企业名称">
      </el-table-column>
      <table-tree-column
        prop="departmentName"
        header-align="center"
        treeKey="id"
        label="部门名称">
      </table-tree-column>
      <el-table-column
        prop="departmentCode"
        header-align="center"
        align="center"
        label="部门代码">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="updateTime"
        header-align="center"
        align="center"
        label="更新时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('enterprise/enterpriseDepartment/update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id,scope.row.enterpriseId)"><i class="el-icon-edit"></i>
          </el-button>
          <el-button v-if="isAuth('enterprise/enterpriseDepartment/delete')" type="text" size="small"
                     @click="deleteHandle(scope.row.id)"><i class="el-icon-delete"></i></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import {treeDataTranslate} from '@/utils'
  import TableTreeColumn from '@/components/table-tree-column'
  import AddOrUpdate from './enterpriseDepartment-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          enterpriseId: '',
          departmentName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      TableTreeColumn,
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/enterprise/enterpriseDepartment/list'),
          method: 'get',
          params: this.$http.adornParams({
            'departmentName': this.dataForm.departmentName,
            'enterpriseId': this.dataForm.enterpriseId
          })
        }).then(({data}) => {
          this.dataList = treeDataTranslate(data.list, 'id', 'parentId')
          this.dataListLoading = false
        })
      },
      // 新增 / 修改
      addOrUpdateHandle (id, enterpriseId) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id, enterpriseId)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/enterprise/enterpriseDepartment/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
