<template>
  <div id="content">
    <div class="row" v-cloak>
      <div class="col-sm-12">
        <div class="ibox float-e-margins" style="margin-bottom: 0px">
          <div class="ibox-title">
            司机列表
          </div>
          <div class="ibox-content" style="padding-bottom: 10px">
            <el-row :gutter="10">
              <el-col :span="4">
                <el-autocomplete
                  class="inline-input"
                  v-model="driverName"
                  :fetch-suggestions="querySearchAsync"
                  placeholder="司机姓名"
                  :trigger-on-focus="false"
                  size="small"
                ></el-autocomplete>
              </el-col>
              <el-col :span="4">
                <el-input size="small" v-model="plateNum" placeholder="车牌号码"></el-input>
              </el-col>
              <el-col :span="4">
                <el-select size="small" clearable v-model="driverStatus" placeholder="司机状态">
                  <el-option label="未绑定车辆" value="0"></el-option>
                  <el-option label="正常运营" value="1"></el-option>
                  <el-option label="合作结束" value="2"></el-option>
                </el-select>
              </el-col>
              <el-col :span="4">
                <el-select size="small" clearable v-model="coModelType" placeholder="合作模式">
                  <el-option label="租赁" value="20"></el-option>
                  <el-option label="租购月供" value="30"></el-option>
                  <el-option label="租购周供" value="40"></el-option>
                  <el-option label="全款" value="10"></el-option>
                </el-select>
              </el-col>
              <el-col :span="4">
                <el-select size="small" clearable v-model="orderBy" placeholder="排序方式">
                  <el-option label="违章扣分" value="score"></el-option>
                  <el-option label="违章罚金" value="money"></el-option>
                  <el-option label="加入时间" value="startDate"></el-option>
                </el-select>
              </el-col>
            </el-row>
            <el-row :gutter="10">
              <el-col :span="4">
                <el-input size="small" v-model="phoneNum" placeholder="司机手机"></el-input>
              </el-col>
              <el-col :span="4">
                <el-select :disabled="branchAble" size="small" clearable v-model="branch" placeholder="选择城市">
                  <el-option label="成都" value="0"></el-option>
                  <el-option label="昆明" value="1"></el-option>
                </el-select>
              </el-col>
            </el-row>
          </div>
          <div class="ibox-footer">
            <el-button icon="search" size="small" @click="query">查询</el-button>
            <el-button :plain="true" @click="reset" type="warning" icon="delete2" size="small">重置</el-button>
            <el-button type="info" size="small" class="pull-right" @click="toAddOrUpdateDriver(0)">新增司机
              <el-icon name="plus"></el-icon>
            </el-button>
          </div>
        </div>
      </div>
    </div>

    <div class="row" v-cloak>
      <div class="col-sm-12">
        <div class="ibox float-e-margins">
          <div class="ibox-title">
            <el-tabs v-model="activeName">
              <el-tab-pane label="全部" name="first"></el-tab-pane>
            </el-tabs>
          </div>
          <div class="ibox-content" element-loading-text="加载中" style="padding-bottom: 50px">
            <el-table
              v-loading.body="isLoading"
              :data="tableData"
              border
              style="width: 100%"
              :stripe="true">
              <el-table-column
                label="司机姓名"
                width="100">
                <template scope="scope">
                  <a @click="toDriverDetail(scope.row.driverId)">{{ scope.row.driverName }}</a>
                </template>
              </el-table-column>
              <el-table-column
                label="司机手机号"
                width="130">
                <template scope="scope">
                  <el-tag v-if="scope.row.phoneStatus=='1'" type="primary">{{ scope.row.phoneNum }}</el-tag>
                  <el-tag v-if="scope.row.phoneStatus=='0'" type="danger">{{ scope.row.phoneNum }}</el-tag>
                  <el-tag v-if="scope.row.phoneStatus=='2'" type="warning">{{ scope.row.phoneNum }}</el-tag>
                  <el-tag v-if="!scope.row.phoneStatus">{{ scope.row.phoneNum }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column
                label="合作模式"
                width="120">
                <template scope="scope">
                  <small>{{ scope.row.coModelType }}</small>
                </template>
              </el-table-column>
              <el-table-column
                label="司机状态"
                width="100">
                <template scope="scope">
                  <span>
                    <el-tag v-if="scope.row.driverStatus=='正常运营'" type="primary">{{scope.row.driverStatus}}</el-tag>
                    <el-tag v-if="scope.row.driverStatus=='合作结束'" type="danger">{{scope.row.driverStatus}}</el-tag>
                    <el-tag v-if="scope.row.driverStatus=='未绑定车辆'">{{scope.row.driverStatus}}</el-tag>
                  </span>
                </template>
              </el-table-column>
              <el-table-column label="车牌号-车型" width="250">
                <template scope="scope">
                  <el-tag @click.native="toAddOrUpdateCar(scope.row.carId)">{{ scope.row.plateNum }}-{{scope.row.carName}}</el-tag>
                </template>
              </el-table-column>
              <el-table-column label="加入时间" width="140">
                <template scope="scope">
                  <small style="margin-left: 10px">{{ scope.row.periodStartDate }}</small>
                </template>
              </el-table-column>
              <el-table-column label="违章情况"
                               width="110">
                <template scope="scope">
                  <small style="margin-left: 10px">{{ scope.row.ticket}}</small>
                </template>
              </el-table-column>
              <el-table-column label="操作"
                               width="100">
                <template scope="scope">
                  <el-button @click="toAddOrUpdateDriver(scope.row.driverId)" :plain="true" size="mini" type="info">
                    查看|修改
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
            <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="pageNum"
              :page-sizes="[10, 50, 100]"
              :page-size="pageSize"
              layout="total, sizes, prev, pager, next"
              :total="total"
              class="pull-right" style="margin-top: 8px">
            </el-pagination>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data () {
      return {
        total: Number,
        pageNum: Number,
        pageSize: Number,
        activeName: 'first',
        plateNum: '',
        driverName: '',
        orderBy: '',
        phoneNum: '',
        driverStatus: '',
        coModelType: '',
        branch: '',
        branchAble: true,
        tableData: [],
        isLoading: false
      }
    },
    methods: {
      fetchData () {
        const _this = this
        this.isLoading = true
        axios.get('/api/manage/driver/list.do', {
          params: {
            branch: this.branch,
            orderBy: this.orderBy,
            plateNum: this.plateNum,
            driverName: this.driverName,
            phoneNum: this.phoneNum,
            driverStatus: this.driverStatus,
            coModelType: this.coModelType,
            pageSize: this.pageSize,
            pageNum: this.pageNum
          }
        }).then(function (res) {
          if (res.data.status === 0) {
            _this.total = res.data.data.total
            _this.tableData = res.data.data.list
            _this.isLoading = false
          } else if (res.data.status === 10) {
            _this.$router.push({name: 'login'})
          }
        })
      },
      toDriverDetail (driverId) {
        this.$router.push({name: 'driver-detail', params: {id: driverId}})
      },
      toAddOrUpdateDriver (driverId) {
        this.$router.push({name: 'add-or-update-driver', params: {id: driverId}})
      },
      toAddOrUpdateCar (carId) {
        this.$router.push({name: 'add-or-update-car', params: {id: carId}})
      },
      query () {
        this.pageNum = 1
        this.pageSize = 10
        this.fetchData()
      },
      handleSizeChange (val) {
        this.pageSize = val
        this.fetchData()
      },
      handleCurrentChange (val) {
        this.pageNum = val
        this.fetchData()
      },
      querySearchAsync (queryString, cb) {
        const _this = this
        axios.get('api/manage/driver/name_list.do', {
          params: {
            driverName: queryString
          }
        }).then(function (res) {
          var driverList = res.data.data
          var results = queryString ? driverList.filter(_this.createFilter(queryString)) : driverList
          // 调用 callback 返回建议列表的数据
          cb(results)
        })
      },
      createFilter (queryString) {
        return (driver) => {
          return (driver.value.indexOf(queryString.toLowerCase()) === 0)
        }
      },
      reset () {
        this.driverName = ''
        this.phoneNum = ''
        this.driverStatus = ''
        this.coModelType = ''
        this.orderBy = ''
        this.plateNum = ''
      }
    },
    watch: {
      driverName (val) {
        sessionStorage.setItem('driverName', val)
      },
      phoneNum (val) {
        sessionStorage.setItem('phoneNum', val)
      },
      driverStatus (val) {
        sessionStorage.setItem('driverStatus', val)
      },
      coModelType (val) {
        sessionStorage.setItem('coModelType', val)
      },
      orderBy (val) {
        sessionStorage.setItem('orderBy', val)
      },
      plateNum (val) {
        sessionStorage.setItem('plateNum', val)
      },
      pageNum (val) {
        sessionStorage.setItem('pageNum', val)
      },
      pageSize (val) {
        sessionStorage.setItem('pageSize', val)
      }
    },
    created: async function () {
      if (this.userBranch === 0) {
        this.branch = '0'
      } else if (this.userBranch === 1) {
        this.branch = '1'
      } else if (this.userBranch === -1) {
        this.branchAble = false
      }
      this.driverName = sessionStorage.getItem('driverName') === null ? '' : sessionStorage.getItem('driverName')
      this.phoneNum = sessionStorage.getItem('phoneNum') === null ? '' : sessionStorage.getItem('phoneNum')
      this.driverStatus = sessionStorage.getItem('driverStatus') === null ? '' : sessionStorage.getItem('driverStatus')
      this.coModelType = sessionStorage.getItem('coModelType') === null ? '' : sessionStorage.getItem('coModelType')
      this.orderBy = sessionStorage.getItem('orderBy') === null ? '' : sessionStorage.getItem('orderBy')
      this.plateNum = sessionStorage.getItem('plateNum') === null ? '' : sessionStorage.getItem('plateNum')
      this.pageNum = sessionStorage.getItem('pageNum') === null ? '' : sessionStorage.getItem('pageNum')
      this.pageSize = sessionStorage.getItem('pageSize') === null ? '' : sessionStorage.getItem('pageSize')
      this.fetchData()
    }
  }
</script>
<style>

  .item {
    margin-top: 10px;
  }

  .el-row {
    margin-bottom: 10px;

  &
  :last-child {
    margin-bottom: 0;
  }

  }
  .el-col {
    border-radius: 4px;
  }

  .grid-content {
    border-radius: 4px;
    min-height: 36px;
  }

  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
</style>
