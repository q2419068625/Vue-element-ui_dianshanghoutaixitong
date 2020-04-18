<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表用户区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['dbbottom',i1=== 0 ?'dbtop':'','vcenter' ]"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag @close="removeRightById(scope.row,item1.id)" closable>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' :'dbtop','vcenter']"
                  v-for="(item2 ,i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      @close="removeRightById(scope.row,item2.id)"
                      closable
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="13">
                    <el-tag
                      type="warning"
                      @close="removeRightById(scope.row,item3.id)"
                      closable
                      v-for="(item3,i3) in item2.children"
                      :key="item3.id"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="30%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区域 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="30%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区域 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="setRightDialogVisible"
      @close="setRightDialogClose"
      width="50%"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightlist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //获取所有角色
      rolelist: [],
      //控制显示添加角色对话框
      addDialogVisible: false,
      //添加角色表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      //添加角色表单验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      //控制显示修改角色对话框
      editDialogVisible: false,
      //修改角色表单数据
      editForm: {},
      //修改角色表单验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      //控制显示分配权限对话框显示与隐藏
      setRightDialogVisible: false,
      //保存所有权限数据
      rightlist: [],
      //树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //默认选中的节点id值素组
      defKeys: [],
      //当前即将分配权限的角色id
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    //获取所有角色列表
    async getRoleList() {
      const { data } = await this.$http.get('roles')
      if (data.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = data.data
      // console.log(this.rolelist);
    },
    //监听添加角色对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮，添加新角色
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        //可以发起添加角色的网络请求
        const { data } = await this.$http.post('roles', this.addForm)
        if (data.meta.status !== 201) {
          return this.$message.error('创建角色失败')
        }
        this.$message.success('创建用户成功')
        this.addDialogVisible = false
        this.getRoleList()
        //    console.log(data);
      })
    },
    //显示修改角色对话框
    async showEditDialog(id) {
      const { data } = await this.$http.get('roles/' + id)
      if (data.meta.status !== 200) {
        return this.$message.error('查询角色信息失败')
      }
      this.editForm = data.data
      this.editDialogVisible = true
    },
    //监听修改角色对话框关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //修改角色信息，并提交
    editRole() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        //发起修改用户角色信息数据请求
        const { data } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if (data.meta.status !== 200) {
          return this.$message, error('修改角色信息失败')
        }
        //刷新用户数据
        this.getRoleList()
        this.$message.success('修改角色信息成功')
        this.editDialogVisible = false
      })
    },
    //删除角色
    async removeRoleById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmResult);
      //如果用户确认删除，则返回值为字符串  confirm
      //如果用户取消删除，则返回值为字符串  cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data } = await this.$http.delete('roles/' + id)
      if (data.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      //更新用户列表
      this.getRoleList()
    },
    //根据id删除对应的全向
    async removeRightById(roleId, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      //如果用户确认删除，则返回值为字符串  confirm
      //如果用户取消删除，则返回值为字符串  cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }

      const { data } = await this.$http.delete(
        `roles/${roleId.id}/rights/${rightId}`
      )
      if (data.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      roleId.children = data.data
    },
    //展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data } = await this.$http.get('rights/tree')
      if (data.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rightlist = data.data
      // console.log(this.rightlist);
      //递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    //通过递归的形式，获取角色下所有三级权限的id 并保存到defkeys数组中
    getLeafKeys(node, arr) {
      //如果当前node属性不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    //监听分配权限管理对话框关闭事件
    setRightDialogClose() {
      this.defKeys = []
    },
    //点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      })
      if (data.meta.status !== 200) {
        this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scope>
.el-tag {
  margin: 7px;
}
.dbtop {
  border-top: 1px solid #eee;
}
.dbbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>