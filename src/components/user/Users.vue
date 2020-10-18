<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">活动管理</a></el-breadcrumb-item>
    </el-breadcrumb>
    <el-card shadow="hover">
      <el-row :gutter="20">
        <el-col :span="9">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="handelClear">
            <el-button slot="append" icon="el-icon-search" @click="handelSearch"></el-button>
          </el-input>
        </el-col>
        <el-col :span="3">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList " style="width: 100%" border>
        <el-table-column type="index">
        </el-table-column>
        <el-table-column prop="username" label="账号" width="250">
        </el-table-column>
        <el-table-column prop="role_name" label="角色" width="250">
        </el-table-column>
        <el-table-column prop="email" label="邮箱" width="250">
        </el-table-column>
        <el-table-column prop="mobile" label="电话" width="250">
        </el-table-column>
        <el-table-column label="状态" width="250">
          <template v-slot:default="scope">
            <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949" @change="switchHandel(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="scope">
           <!-- 修改用户 -->
          <el-button type="primary" icon="el-icon-edit" size="mini" @click = "editHandel(scope.row.id)"></el-button>
          <!-- 删除用户 -->
          <el-button type="danger" icon="el-icon-delete" size="mini" @click = "deleteHandel(scope.row.id)"></el-button>
          <el-tooltip :enterable="false" effect="dark" content="分配角色" placement="top">
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
          </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum"
      :page-sizes="[1,2,5,10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="40%" center>
      <!-- 添加用户界面 -->
      <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="账号" prop="username">
          <el-input v-model="ruleForm.username" ></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="ruleForm.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="checkPass">
          <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="ruleForm.email"></el-input>
        </el-form-item>
        <el-form-item label="号码" prop="mobile">
          <el-input v-model="ruleForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
          <el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
      </span>
    </el-dialog>
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="40%" center>
      <!-- 修改用户界面 -->
      <el-form :model="currentUser" status-icon :rules="rules" ref="editForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="账号" prop="username">
          <el-input v-model="currentUser.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="currentUser.email"></el-input>
        </el-form-item>
        <el-form-item label="号码" prop="mobile">
          <el-input v-model="currentUser.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
          <el-button @click="editDialogVisible=false" @close= "closeEditForm('editForm')">取消</el-button>
          <el-button type="primary" @click="submitEditForm('editForm')">提交</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'))
      } else {
        if (this.ruleForm.pass !== '') {
          this.$refs.ruleForm.validateField('checkPass')
        }
        callback()
      }
    }
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.ruleForm.password) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    var validateEmail = (rule, value, callback) => {
      const reg = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+$/
      if (value === '') {
        callback()
      } else if (!reg.test(value)) {
        callback(new Error('请输入正确的邮箱'))
      } else {
        callback()
      }
    }
    var validatePhone = (rule, value, callback) => {
      const reg = /^\d{11}$/
      if (value === '') {
        callback()
      } else if (!reg.test(value)) {
        callback(new Error('请输入11位号码'))
      } else {
        callback()
      }
    }
    return {
      ruleForm: {
        username: '',
        password: '',
        checkPass: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [{
          required: true,
          validator: validatePass,
          trigger: 'blur'
        }],
        checkPass: [{
          required: true,
          validator: validatePass2,
          trigger: 'blur'
        }],
        email: [{
          validator: validateEmail,
          trigger: 'blur'
        }],
        mobile: [{
          validator: validatePhone,
          trigger: 'blur'
        }]
      },
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      userList: [],
      currentUser: {
        username: '',
        email: '',
        mobile: ''
      },
      total: 0,
      dialogVisible: false,
      editDialogVisible: false
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const {
        data: res
      } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (event) {
      this.queryInfo.pagesize = event
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    handleCurrentChange (currenPage) {
      this.queryInfo.pagenum = currenPage
      this.getUserList()
    },
    async switchHandel (scopeSwitch) {
      const {
        data: res
      } = await this.$http.put(`users/${scopeSwitch.id}/state/${scopeSwitch.mg_state}`)
      if (res.meta.status !== 200) {
        scopeSwitch.mg_state = !scopeSwitch.mg_state
        return this.$message.error('用户状态更新失败')
      }
      this.$message.success('用户状态更新成功')
    },
    async editHandel (id) {
      const { data: res } = await this.$http.get(`users/${id}`)
      this.currentUser = res.data
      this.editDialogVisible = true
    },
    handelSearch () {
      this.getUserList()
    },
    handelClear () {
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    submitForm (formName) {
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          const { data: res } = await this.$http.post('users', this.ruleForm)
          if (res.meta.status !== 201) {
            this.$message.error(res.meta.msg)
          } else {
            this.$message.success(res.meta.msg)
            this.dialogVisible = false
            this.getUserList()
          }
          console.log(res)
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    submitEditForm (formName) {
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          const { data: res } = await this.$http.put(`users/${this.currentUser.id}`, this.currentUser)
          if (res.meta.status !== 200) {
            this.$message.error(res.meta.msg)
          } else {
            this.$message.success(res.meta.msg)
            this.editDialogVisible = false
            this.getUserList()
          }
          console.log(res)
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    closeEditForm (formName) {
      this.resetForm(formName)
    },
    resetForm (formName) {
      this.$refs[formName].resetFields()
    },
    deleteHandel (id) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`users/${id}`)
        if (res.meta.status !== 200) {
          this.$message.error(res.meta.msg)
        } else {
          this.$message({
            type: 'success',
            message: res.meta.msg
          })
          this.getUserList()
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }

  }

}
</script>

<style lang="less" scoped>
</style>
