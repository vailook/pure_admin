<template>
  <div class="JNPF-common-layout">

    <div class="JNPF-common-layout-center">
      <el-row class="JNPF-common-search-box" :gutter="16">
        <el-form @submit.native.prevent>
          <el-col :span="6">
            <el-form-item label="客户">
              <el-input v-model="query.customerName" placeholder="请输入" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="发货日期">
              <el-date-picker v-model="query.deliveryDate" type="daterange"
                              value-format="timestamp" format="yyyy-MM-dd" start-placeholder="开始日期"
                              end-placeholder="结束日期">
              </el-date-picker>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="FRP规格">
              <el-input v-model="query.frpSpec" placeholder="请输入" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item>
              <el-button type="primary" icon="el-icon-search" @click="search()">查询</el-button>
              <el-button icon="el-icon-refresh-right" @click="reset()">重置</el-button>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
      <div class="JNPF-common-layout-main JNPF-flex-main">
        <div class="JNPF-common-head">
          <div>
            <el-button type="primary" icon="el-icon-plus" v-if="!isCustomer"
                       @click="addOrUpdateHandle()">新增
            </el-button>
          </div>
          <div class="JNPF-common-head-right">
            <el-tooltip effect="dark" content="刷新" placement="top">
              <el-link icon="icon-ym icon-ym-Refresh JNPF-common-head-icon" :underline="false"
                       @click="reset()"/>
            </el-tooltip>
            <screenfull isContainer/>
          </div>
        </div>
        <JNPF-table v-loading="listLoading" :data="list" @sort-change='sortChange'>
          <el-table-column prop="customerName" label="客户" width="0" align="left"
          />
          <el-table-column prop="deliveryDate" label="发货日期" width="0" align="left"
          />
          <el-table-column prop="frpSpec" label="FRP规格" width="0" align="left"
          />
          <el-table-column prop="deliveryBoxQty" label="发货木箱" width="0" align="left"
          />
          <el-table-column prop="deliveryTubeQty" label="发货管子数量" width="0" align="left"
          />
          <el-table-column label="操作" fixed="right"
                           width="100">
            <template slot-scope="scope">
              <el-button type="text" v-if="!isCustomer && scope.row.status === 1"
                         @click="addOrUpdateHandle(scope.row.id)">编辑
              </el-button>
              <el-button type="text" class="JNPF-table-delBtn" v-if="!isCustomer && scope.row.status === 1"
                         @click="handleDel(scope.row.id)">删除
              </el-button>
              <el-button type="text" style="color:green" v-if="isCustomer && scope.row.status === 1"
                         @click="handleConfirm(scope.row.id)">确认
              </el-button>
              <span v-if="scope.row.status === 2" style="color:#af39b1">已确认</span>
            </template>
          </el-table-column>
        </JNPF-table>
        <pagination :total="total" :page.sync="listQuery.currentPage" :limit.sync="listQuery.pageSize"
                    @pagination="initData"/>
      </div>
    </div>
    <JNPF-Form v-if="formVisible" ref="JNPFForm" @refresh="refresh"/>
 
  </div>
</template>

<script>
import request from '@/utils/request'
// import {getDictionaryDataSelector} from '@/api/systemData/dictionary'
import JNPFForm from './Form'

// import {previewDataInterface} from '@/api/systemData/dataInterface'

export default {
  components: {JNPFForm},
  data() {
    return {
      query: {
        customerName: undefined,
        deliveryDate: undefined,
        frpSpec: undefined,
      },
      treeProps: {
        children: 'children',
        label: 'fullName',
        value: 'id'
      },
      isCustomer: false,
      list: [],
      listLoading: true,
      total: 0,
      listQuery: {
        currentPage: 1,
        pageSize: 20,
        sort: "desc",
        sidx: "",
      },
      formVisible: false,
      exportBoxVisible: false,
      columnList: [
        {prop: 'customerCode', label: '客户Code'},
        {prop: 'customerName', label: '客户'},
        {prop: 'deliveryDate', label: '发货日期'},
        {prop: 'frpSpec', label: 'FRP规格'},
        {prop: 'deliveryBoxQty', label: '发货木箱'},
        {prop: 'deliveryTubeQty', label: '发货管子数量'},
      ],
    }
  },
  computed: {},
  created() {
    this.initData()
    this.initIsCustomer();
  },
  methods: {
    sortChange({column, prop, order}) {
      this.listQuery.sort = order == 'ascending' ? 'asc' : 'desc'
      this.listQuery.sidx = !order ? '' : prop
      this.initData()
    },
    initIsCustomer() {
      request({
        url: '/api/project/CusDelivery/isCustomer',
        method: 'get'
      }).then(res => {
        this.isCustomer = res.data;
      })
    },
    initData() {
      this.listLoading = true;
      let _query = {
        ...this.listQuery,
        ...this.query
      };
      request({
        url: `/api/project/CusDelivery/getList`,
        method: 'post',
        data: _query
      }).then(res => {
        var _list = [];
        for (let i = 0; i < res.data.list.length; i++) {
          let _data = res.data.list[i];
          _list.push(_data)
        }
        this.list = _list
        this.total = res.data.pagination.total

        this.listLoading = false
      })
    },
    handleDel(id) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        type: 'warning'
      }).then(() => {
        request({
          url: `/api/project/CusDelivery/${id}`,
          method: 'DELETE'
        }).then(res => {
          this.$message({
            type: 'success',
            message: res.msg,
            onClose: () => {
              this.initData()
            }
          });
        })
      }).catch(() => {
      });
    },
    handleConfirm(id) {
      request({
        url: `/api/project/CusDelivery/confirm/${id}`,
        method: 'PUT'
      }).then(res => {
        this.$message({
          type: 'success',
          message: res.msg,
          onClose: () => {
            this.initData()
          }
        });
      })
    },
    addOrUpdateHandle(id, isDetail) {
      this.formVisible = true
      this.$nextTick(() => {
        this.$refs.JNPFForm.init(id, isDetail)
      })
    },


    search() {
      this.listQuery = {
        currentPage: 1,
        pageSize: 20,
        sort: "desc",
        sidx: "",
      }
      this.initData()
    },
    refresh(isrRefresh) {
      this.formVisible = false
      if (isrRefresh) this.reset()
    },
    reset() {
      for (let key in this.query) {
        this.query[key] = undefined
      }
      this.listQuery = {
        currentPage: 1,
        pageSize: 20,
        sort: "desc",
        sidx: "",
      }
      this.initData()
    }
  }
}
</script>
