<template>
  <div class="servers">
    <div class='top_part'>
       <header-bar :noServer = true></header-bar>
    </div>
    <div class='cont_part'>
      <P class='back' @click='back'><i class='el-icon-arrow-left'></i>返回</P>
      <div class='table_part'>
        <p class='title'>
          <span>服务管理</span>
          <i class='el-icon-plus' @click='showLog = true' ></i>
        </p>
        <el-table
          :data="server"
          style="width: 100%">
          <el-table-column
            label="服务名称"
            prop='name'
            width='120'
            >
          </el-table-column>
          <el-table-column
            label="BrokerURL地址"
            prop='brokerUrl'>
          </el-table-column>
          <el-table-column
            label="WebaseURL地址"
            prop='webaseUrl'>
          </el-table-column>
          <el-table-column
            width='100'
            label="操作">
            <template slot-scope='scope'>
              <i class='el-icon-edit table_icon' @click='adit(scope.row)' title='编辑'></i>
              <i class='el-icon-delete table_icon' @click='deleteItem(scope.row)' title='删除'></i>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
    <el-dialog :title="title" :visible.sync="showLog">
      <el-form :model="form" :rules="rules" ref='form'>
        <el-form-item label="名称:" prop='name'>
          <el-input v-model.trim="form.name" autocomplete="off" placeholder="请输入服务名称(1-20位字母数字下划线)"></el-input>
        </el-form-item>
        <el-form-item label="BrokerURL地址:" prop='brokerUrl'>
          <el-input v-model.trim="form.brokerUrl" autocomplete="off"  placeholder="例如: 'http://127.0.0.1:8080/weevent'"></el-input>
        </el-form-item>
        <el-form-item label="WebaseURL地址:" prop='webaseUrl'>
          <el-input v-model.trim="form.webaseUrl" autocomplete="off"  placeholder="例如: 'http://127.0.0.1:8080/webase'"></el-input>
        </el-form-item>
        <el-form-item label="授权用户:" v-show='showAccount'>
          <el-select
            v-model="form.userIdList"
            multiple
            placeholder="请选择用户">
            <el-option
              v-for="item in accountList"
              :key="item.id"
              :label="item.username"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label='配置规则引擎:'>
          <el-radio-group v-model='form.isConfigRule'>
            <el-radio label='1'>是</el-radio>
            <el-radio label='2'>否</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showLog = false">取 消</el-button>
        <el-button type="primary" @click="confirm">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import API from '../API/resource.js'
import headerBar from '../components/headerBar'
export default {
  components: {
    headerBar
  },
  data () {
    var checkName = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入用户名'))
      } else {
        let regex = /^[0-9A-Za-z]{1,20}$/
        if (regex.exec(value)) {
          callback()
        } else {
          callback(new Error('服务名格式错误'))
        }
      }
    }
    var checkBroker = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('服务端口不能为空'))
      } else {
        let data = {
          'userId': parseInt(localStorage.getItem('userId')),
          'id': parseInt(localStorage.getItem('brokerId')),
          'brokerUrl': value
        }
        API.checkBrokerServer(data).then(res => {
          if (res.data === 'SUCCESS') {
            callback()
          } else {
            callback(new Error('服务地址错误,无法连接!'))
          }
        })
      }
    }
    var checkWeBase = (rule, value, callback) => {
      if (value === '') {
        callback()
      } else {
        // let data = {
        //   'userId': parseInt(localStorage.getItem('userId')),
        //   'id': parseInt(localStorage.getItem('brokerId')),
        //   'weBaseUrl': value
        // }
        // API.checkBrokerServer(data).then(res => {
        //   if (res.data === 'SUCCESS') {
        //     callback()
        //   } else {
        //     callback('服务地址错误,服务无法连接')
        //   }
        // })
        callback()
      }
    }
    return {
      server: [],
      showLog: false,
      isEdit: false,
      title: '新增服务',
      accountList: [],
      showAccount: true,
      form: {
        name: '',
        brokerUrl: '',
        webaseUrl: '',
        userIdList: [],
        isConfigRule: '1'
      },
      brokerId: '',
      rules: {
        name: [
          { validator: checkName, trigger: 'blur' }
        ],
        brokerUrl: [
          { validator: checkBroker, trigger: 'blur' }
        ],
        webaseUrl: [
          { validator: checkWeBase, trigger: 'blur' }
        ]
      }
    }
  },
  watch: {
    showLog (nVal) {
      if (!nVal) {
        this.title = '新增服务'
        this.$refs.form.resetFields()
        this.$set(this.form, 'name', '')
        this.$set(this.form, 'brokerUrl', '')
        this.$set(this.form, 'webaseUrl', '')
        this.$set(this.form, 'userIdList', [])
        this.brokerId = ''
        this.showAccount = true
      } else {
        API.accountList('').then(res => {
          if (res.data.status === 200) {
            this.accountList = [].concat(res.data.data)
          }
        })
      }
    }
  },
  mounted () {
    this.getServer()
  },
  methods: {
    confirm () {
      let vm = this
      vm.$refs.form.validate((valid) => {
        if (valid) {
          if (vm.isEdit) {
            vm.editServer()
          } else {
            vm.addServer()
          }
        } else {
          return false
        }
      })
    },
    addServer () {
      let data = {
        name: this.form.name,
        brokerUrl: this.form.brokerUrl,
        webaseUrl: this.form.webaseUrl,
        userId: Number(localStorage.getItem('userId')),
        isConfigRule: this.form.isConfigRule
      }
      data.userIdList = [].concat(this.form.userIdList)
      API.addServer(data).then(res => {
        if (res.status === 200) {
          if (res.data.status === 200) {
            this.$message({
              type: 'success',
              message: '新增成功'
            })
            this.getServer()
          } else {
            this.$message({
              type: 'warning',
              message: '新增失败'
            })
          }
        } else {
          this.$message({
            type: 'warning',
            message: '新增失败'
          })
        }
        this.showLog = false
      })
    },
    getServer () {
      API.getServer('').then(res => {
        if (res.status === 200) {
          this.server = [].concat(res.data)
        }
      })
    },
    editServer () {
      this.isEdit = false
      let data = {
        name: this.form.name,
        brokerUrl: this.form.brokerUrl,
        webaseUrl: this.form.webaseUrl,
        id: this.brokerId,
        userId: Number(localStorage.getItem('userId')),
        isConfigRule: this.form.isConfigRule
      }
      data.userIdList = [].concat(this.form.userIdList)
      API.updateServer(data).then(res => {
        if (res.status === 200) {
          if (res.data.status === 200) {
            this.$message({
              type: 'success',
              message: '编辑成功'
            })
            this.getServer()
          } else {
            this.$message({
              type: 'warning',
              message: '编辑失败'
            })
          }
        } else {
          this.$message({
            type: 'warning',
            message: '编辑失败'
          })
        }
        this.showLog = false
      })
    },
    adit (e) {
      let vm = this
      vm.$set(this.form, 'name', e.name)
      vm.$set(this.form, 'brokerUrl', e.brokerUrl)
      vm.$set(this.form, 'webaseUrl', e.webaseUrl)
      vm.$set(this.form, 'userIdList', [])
      vm.brokerId = e.id
      if (e.isCreator === '1') {
        let data = {
          brokerId: e.id
        }
        API.permissionList(data).then(res => {
          if (res.data.status === 200) {
            res.data.data.forEach(e => {
              vm.form.userIdList.push(e.userId)
            })
          }
        })
      } else {
        this.showAccount = false
      }
      this.title = '编辑服务信息'
      this.showLog = true
      this.isEdit = true
    },
    deleteItem (e) {
      var vm = this
      vm.$confirm('确认删除该服务？').then(_ => {
        let data = {
          'id': e.id
        }
        API.deleteServer(data).then(res => {
          if (res.data.status === 200) {
            vm.$message({
              type: 'success',
              message: '删除成功'
            })
            vm.getServer()
            if (e.id === parseInt(localStorage.getItem('brokerId'))) {
              if (vm.server.length > 0) {
                let newId = vm.server[0].id
                localStorage.setItem('brokerId', newId)
              } else {
                localStorage.removeItem('brokerId')
              }
            }
          } else {
            this.$message({
              type: 'warning',
              message: '删除操作失败'
            })
          }
        })
      })
    },
    back () {
      this.$router.go(-1)
    }
  }
}
</script>
