<template>
  <div class="add-nav-warp">
    <!-- 添加网站popup -->
    <el-dialog
      title="添加网站"
      :visible.sync="show"
      width="320"
      @close="$emit('update:show', false)"
    >
      <el-form ref="ruleForm" label-width="100px" :model="form" :rules="rules">
        <el-form-item label="网站链接" prop="url">
          <el-input placeholder="http://www.baidu.com/" v-model="form.url"  :disabled="type === 'update'" />
        </el-form-item>

        <el-form-item label="网站分类" prop="categoryId">
          <el-select v-model="form.categoryId" placeholder="请选择">
            <el-option-group
              v-for="group in categorys"
              :key="group._id"
              :label="group.name"
            >
              <el-option
                v-for="item in group.children"
                :key="item._id"
                :label="item.name"
                :value="item._id"
              ></el-option>
            </el-option-group>
          </el-select>
        </el-form-item>
        <el-form-item label="网站名称" prop="name" v-if="isError">
          <el-input placeholder="输入网站名称" v-model="form.name" />
        </el-form-item>
        <el-form-item label="网站描述" prop="desc" v-if="isError">
          <el-input placeholder="输入网站描述" v-model="form.desc" />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="addNav('ruleForm')">
            提交
          </el-button>
          <p>提交后爬虫会自动补全网站信息</p>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  props: {
    show: {
      type: Boolean,
      default: false,
    },
    type: {
      validator: function(value = 'create') {
        // 这个值必须匹配下列字符串中的一个
        return ['create', 'update'].indexOf(value) !== -1
      },
    },
    item: Object,
  },
  data() {
    return {
      categorys: [],
      form: {
        url: '',
        categoryId: '',
        name: '',
        desc: '',
      },
      rules: {
        url: [
          { required: true, message: '请输入url', trigger: 'blur' },
          {
            pattern: /(http|ftp|https):\/\/[\w\-_]+(\.[\w\-_]+)+([\w\-\.,@?^=%&:/~\+#]*[\w\-\@?^=%&/~\+#])?/,
            message: '请输入正确的url',
          },
        ],
        categoryId: [
          { required: true, message: '请选择分类', trigger: 'blur' },
        ],
        name: [],
        desc: [],
      },
      isError: false,
    }
  },
  methods: {
    async getCategorys() {
      const { data } = await this.$api.getCategoryList()
      this.categorys = data
    },
    async addNav(formName) {
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          // 判断编辑还是更新
          if (this.type === 'update') {
            const id = this.item._id
            const res = await this.$api.editNav({
              id,
              ...this.form
            })
          } else {
            const res = await this.$api.addAudit(this.form)
            if (res.msg) {
              this.$message.error(`${res.msg}，请手动输入！`)
              this.isError = true
              return false
            }
            this.$message(`感谢您的提交，请等待后台审核通过！`)
          }
          this.form = {}

          this.$emit('update:show', false)
          this.$emit('submit')
        } else {
          return false
        }
      })
    },
  },
  watch: {
    item(ite) {
      this.isError = true
      this.form.name = ite.name
      this.form.url = ite.href
      this.form.desc = ite.desc
      this.form.categoryId = ite.categoryId
    },
  },
  created() {
    this.getCategorys()
  },
}
</script>

<style lang="scss" scoped>

.el-icon-folder-add {
  margin-right: 5px;
}

.el-dialog {
  min-width: 320px;
}
</style>
