<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          :tab-position="'right'"
          @tab-click="tabClicked"
          v-model="activeIndex"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="所属商品分类">
              <el-input  disabled style="width:400px" v-model="cat_name"></el-input>
            </el-form-item>
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"  style="width:75%"></el-input>
            </el-form-item>
            <el-form-item label="商品价格(元)" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number" style="width:400px"></el-input>
            </el-form-item>
            <el-form-item label="商品重量(kg)" prop="goods_weight">
              <el-input
                v-model="editForm.goods_weight"
                type="number"
                style="width:400px"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input
                v-model="editForm.goods_number"
                type="number"
                style="width:400px"
              ></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的item项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
              <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
              <!-- 图片上传 upload接口 -->
            <!-- action表示图片要上传到的后台API地址 -->
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="editForm.goods_introduce"></quill-editor>
          </el-tab-pane>
          <el-tab-pane label="提交修改" name="5">
            <el-button type="primary" @click="edit">确认提交修改</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      // 接收编程式导航传过来的商品id
      goods_id: 0,
      editForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_introduce: '',
        goods_cat: '',
        // 上传图片的临时路径对象
        pics: {},
        // 商品的参数数组
        attrs: [],
      },
      // 商品所属的分类id
      cat_id: {
        cat_one_id: 0,
        cat_two_id: 0,
        cat_three_id: 0,
      },
      // 商品所属分类名称
      cat_name: '',
      // 商品信息表单验证规则
      editFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
      },
      // 动态参数列表数据
      manyTableData: [],
      // 静态属性列表数据
      onlyTableData: [],
      // 上传图片的url地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token'),
      },
      // 要预览的图片路径
      previewPath: '',
      // 控制图片预览对话框的显示与隐藏
      previewVisible: false,
    }
  },
  created() {
    this.editGoodsById()
  },
  methods: {
    // 根据id查询商品信息
    async editGoodsById() {
      this.goods_id = parseInt(this.$route.query.goods_id)
      // console.log(this.goods_id)
      const { data: res } = await this.$http.get('goods/' + this.goods_id)
      console.log(res.data)
      if (res.meta.status !== 200) {
        return this.$message.error('获取该商品信息失败!')
      }
      this.editForm.goods_name = res.data.goods_name
      this.editForm.goods_price = res.data.goods_price
      this.editForm.goods_weight = res.data.goods_weight
      this.editForm.goods_number = res.data.goods_number
      this.editForm.goods_cat = res.data.goods_cat
      this.editForm.goods_introduce = res.data.goods_introduce
      this.editForm.pics = res.data.pics
      this.cat_id.cat_one_id = res.data.cat_one_id
      this.cat_id.cat_two_id = res.data.cat_two_id
      this.cat_id.cat_three_id = res.data.cat_three_id
      console.log(this.editForm)
      console.log(this.cat_id)
      console.log(this.cat_id.cat_one_id)
      const { data: req } = await this.$http.get(
        `categories/${this.cat_id.cat_one_id}`
      )
      if (req.meta.status !== 200) {
        return this.$message.error('获取分类名称失败!')
      }
      const cate_name1 = req.data.cat_name
      const { data: rex } = await this.$http.get(
        `categories/${this.cat_id.cat_two_id}`
      )
      if (rex.meta.status !== 200) {
        return this.$message.error('获取分类名称失败!')
      }
      const cate_name2 = rex.data.cat_name
      const { data: rea } = await this.$http.get(
        `categories/${this.cat_id.cat_three_id}`
      )
      if (rea.meta.status !== 200) {
        return this.$message.error('获取分类名称失败!')
      }
      const cate_name3 = rea.data.cat_name
      this.cat_name = cate_name1 + ' / ' + cate_name2 + ' / ' + cate_name3
    },
    // tabs页签点击事件
    async tabClicked() {
      console.log(this.activeIndex)
      // 证明访问的动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' },
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数列表失败!')
        }
        // console.log(res.data)
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      }
      if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' },
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性列表失败!')
        }
        // console.log(res.data)
        this.onlyTableData = res.data
      }
      if (this.activeIndex === '3') {
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      // file即将要预览的图片的信息
      // console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除图片的操作
    handleRemove(file) {
      // file为将要被移除的图片的信息,包含response属性
      console.log(file)
      // 1. 获取即将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2. 从pics数组中，找到这个图片对应的索引值 (x表示数组中的每一项，i为对应项的索引值)
      const i = this.addForm.pics.findIndex((x) => x.pic === filePath)
      // 3. 调用数组的splice方法，把图片信息对象，从pics数组中移除
      this.addForm.pics.splice(i, 1)
      // console.log(this.addForm)
    },
    // 监听图片上传成功的事件
    handleSuccess(response) {
      console.log(response)
      // console.log(response) response:图片上传成功返回的数据对象
      // 1. 拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2. 将图片信息对象，push到pics数组中
      this.editForm.pics.push(picInfo)
      console.log(this.editForm)
    },
    // 点击按钮，确认修改商品信息提交事件
    edit() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('请将必填项填写完整!')
        // 处理动态参数
        console.log(this.manyTableData)
        console.log(this.onlyTableData)
        this.manyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' '),
          }
          this.editForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach((item) => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.editForm.attrs.push(newInfo)
        })
        console.log(this.editForm)
        const { data: res } = await this.$http.put(
          'goods/' + this.goods_id,
          this.editForm
        )
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('修改该商品信息失败!')
        }
        this.$router.push('/goods')
      })
    },
  },
  computed: {
    cateId() {
      const arr = this.editForm.goods_cat.split(',')
      if (arr.length === 3) {
        return arr[2]
      }
    },
  },
}
</script>

<style lang="less" scoped>
.el-tabs {
  margin: 10px 100px 10px 50px;
}
.el-checkbox {
  margin: 5px 10px 0 0 !important;
}
.el-form-item {
  margin-bottom: 10px;
}
.el-form-item__label {
  padding: 0 0 5px !important;
}
</style>