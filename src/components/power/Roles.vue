<template>
<div>
    <!--面包屑导航区域-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!--卡片视图-->
        <el-card>
             <!--添加角色按钮区-->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!--角色列表区域-->
            <el-table :data="rolelist" border stripe>
                <el-table-column type="expand">
                    <template v-slot="scope">
                      <el-row :class="['bdbottom','vcenter',i1 === 0 ? 'bdtop' : '']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                          <!--渲染一级权限-->
                          <el-col :span="5">
                              <el-tag closable @close="removeRightById(scope.row,item1.id)" >{{item1.authName}}</el-tag>
                              <i class="el-icon-caret-right"></i>
                          </el-col>
                          <!--渲染二级和三级权限-->
                          <el-col :span="19">
                              <!--通过for循环 嵌套渲染二级权限-->
                              <el-row :class="[i2 === 0 ? '':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                                  <el-col :span="6">
                                       <el-tag  closable @close="removeRightById(scope.row,item2.id)"  type="success">{{item2.authName}}</el-tag>
                                       <i class="el-icon-caret-right"></i>
                                  </el-col>
                                  <el-col :span="18">
                                      <el-tag  closable @close="removeRightById(scope.row,item3.id)" type="warning" v-for="item3 in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                                  </el-col>
                              </el-row>
                          </el-col>
                      </el-row>
                    </template>
                </el-table-column>
                <!--索引列-->
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc" ></el-table-column>
                <el-table-column label="操作" width=300px>
                    <template  v-slot="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!--分配权限的对话框-->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%">
            <!--树形控件-->
            <el-tree  ref="treeRef" :default-checked-keys="defKeys" default-expand-all show-checkbox node-key="id" :data="rightslist" :props="treeProps" ></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights" >确 定</el-button>
            </span>
        </el-dialog>
        <!--添加角色的对话框-->
        <el-dialog
          title="添加角色"
          :visible.sync="addRoleDialogVisible"
          width="50%"
          @close="addDialogClosed">
          <!--内容主题区域-->
          <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" >
            <el-form-item label="角色名称" prop="roleName">
              <el-input v-model="addForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" >
              <el-input v-model="addForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <!--底部区域-->
          <span slot="footer" class="dialog-footer">
            <el-button @click="addRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addRole">确 定</el-button>
          </span>
        </el-dialog>
        <!--修改角色对话框-->
        <el-dialog
          title="修改角色"
          :visible.sync="editRoleDialogVisible"
          width="50%"
          @close="editRoleDialogClosed"
          >
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px" >
            <el-form-item label="角色名称" >
              <el-input v-model="editForm.roleName"></el-input>
            </el-form-item>
             <el-form-item label="角色描述">
              <el-input v-model="editForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRoleInfo">确 定</el-button>
          </span>
        </el-dialog>
</div>
</template>
<script>
export default {
  data () {
    return {
      // 所有角色列表
      rolelist: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选择的节点id值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 控制添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur' }
        ]
      },
      editRoleDialogVisible: false,
      // 编辑用户的表单数据
      editForm: {},
      // 编辑表单的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur' }
        ]
      }

    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    // 根据id删除对于的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    // 通过递归的形式 获取角色下所有三级权限的id，并保存到defkeys数组中
    getLeafKeys (node, arr) {
      // 如果当前node 节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      // 把获取到的权限数据保存到data中
      this.rightslist = res.data
      console.log(this.rightslist)
      // 递归获取三级节点的id
      this.defKeys = []
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 点击按钮，添加角色
    addRole () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        // 隐藏添加角色的对话框
        this.addRoleDialogVisible = false
        // 重新获取角色列表
        this.getRolesList()
      })
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 根据id删除对应信息
    async removeRoleById (id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      // 如果用户确认删除，则返回字符串confirm
      // 如果用户取消删除，则返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('删除角色失败')
      }
      this.$message.success('删除角色成功')
      this.getRolesList()
    },
    // 监听修改角色对话框的关闭事件
    editRoleDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 展示角色编辑的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败')
      }
      this.editForm = res.data
      this.editRoleDialogVisible = true
    },
    // 修改修改用户信息并提交
    editRoleInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        console.log(this.editForm)
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        console.log(res)
        if (res.meta.status !== 200) {
          this.$message.error('修改角色失败')
        }
        // 隐藏添加用户的对话框
        this.editRoleDialogVisible = false
        this.$message.success('修改角色成功')
        // 重新获取用户列表
        this.getRolesList()
      })
    }

  }
}
</script>
<style lang="less">
  .el-tag{
      margin: 7px
  }
  .bdtop {
      border-top: 1px solid #eeeeee
  }
  .bdbottom {
       border-bottom: 1px solid #eeeeee
  }
  .vcenter {
      display: flex;
      align-items: center
  }

</style>
