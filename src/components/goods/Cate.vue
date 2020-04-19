<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加分类区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格主体内容区域 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="showEditCateDialog(scope.row)"
          >编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeCate(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <!-- 添加分类对话框 -->
      <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed"
      >
        <el-form
          :model="addCateForm"
          :rules="addCateFormRules"
          ref="addCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类：">
            <!--options绑定的数据源  -->
            <el-cascader
              expand-trigger="hover"
              v-model="selectKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="selectCateChange"
              clearable
              change-on-select
              visible-change
            ></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改分类对话框 -->
      <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%">
        <el-form
          :model="editCateForm"
          :rules="editCateFormRules"
          ref="editCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="editCateForm.cat_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editCate">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //商品分类数据列表，默认为空
      catelist: [],
      //总数据条数
      total: 0,
      //为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //表示将当前定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //表示将当前定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          //表示将当前定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      //控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      //添加分类表单对象
      addCateForm: {
        //父级分类id
        cat_pid: 0,
        //将要添加分类的名称
        cat_name: '',
        //分类的等级，默认要添加的等级为一级
        cat_level: 0
      },
      //添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //父级分类数据列表
      parentCateList: [],
      //父级选择的id数组
      selectKeys: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //控制显示修改分类对话框显示与隐藏
      editCateDialogVisible: false,
      //修改分类表单对象
      editCateForm: {
        //父级分类id
        cat_pid: 0,
        //将要添加分类的名称
        cat_name: '',
        //分类的等级，默认要添加的等级为一级
        cat_level: 0,
        cat_id:0
      },
      //修改分类表单的验证规则对象
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取商品分类数据
    async getCateList() {
      const { data } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      // console.log(data.data)
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      //把数据列表复制给 catelist
      this.catelist = data.data.result
      //为总数据条数复制
      this.total = data.data.total
    },
    //监听pagesize改变
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getCateList()
    },
    //监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    //点击显示添加分类对话框
    showAddCateDialog() {
      //先获取父级数据
      this.getParentCateList()
      //展示对话框
      this.addCateDialogVisible = true
    },
    //获取父级分类的数据列表
    async getParentCateList() {
      const { data } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (data.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败')
      }
      // console.log(data.data);
      this.parentCateList = data.data
    },
    //选择项发生变化触发这个函数
    selectCateChange() {
      //  console.log(this.selectKeys);
      //如果selectKeys数组中的length大于0证明选父级分类
      //反之，就说明没有选中任何父级分类
      if (this.selectKeys.length > 0) {
        //父级分类的id
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        //为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectKeys.length
        return
      } else {
        //父级分类的id
        this.addCateForm.cat_pid = 0
        //为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    //点击按钮，添加新分类
    addCate() {
      //  console.log(this.addCateForm);
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data } = await this.$http.post('categories', this.addCateForm)
        if (data.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    //监听对话框关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    //点击按钮显示修改分类对话框
    async showEditCateDialog(id) {
      //先获取对应id的数据
      //  console.log(id);
      const { data } = await this.$http.get('categories/' + id.cat_id)

      if (data.meta.status !== 200) {
        return this.$message.error('查询分类失败')
      }
      // console.log(data);
      this.editCateForm = data.data
      this.editCateDialogVisible = true
    },
    //点击按钮，修改分类
    editCate(){
      this.$refs.editCateFormRef.validate(async valid =>{
        if(!valid) return
        // console.log(this.editCateForm);
        const {data} =  await this.$http.put('categories/' +this.editCateForm.cat_id,{
          cat_name:this.editCateForm.cat_name
        })
        if(data.meta.status !== 200){
          this.$message.error('修改分类失败')
        }
        this.$message.success('修改分类成功')
        this.getCateList()
        this.editCateDialogVisible = false
     })
    },
    //点击按钮删除分类
    async removeCate(id){
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
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
      if(confirmResult !== 'confirm'){
        this.$message.info('取消删除分类')
      }
      // console.log(id);
      const {data} =  await this.$http.delete('categories/'+id)
      if(data.meta.status !==200){
        return this.$message.error('删除分类失败')
      }
      this.$message.success('删除分类成功')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>