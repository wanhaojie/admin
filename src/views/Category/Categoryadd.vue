<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home/index' }"
        >首页</el-breadcrumb-item
      >
      <el-breadcrumb-item :to="{ path: '/home/category' }"
        >商品分类列表</el-breadcrumb-item
      >
      <el-breadcrumb-item>{{ toptitle }}</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 表单 -->
    <el-form
      :model="ruleForm"
      :rules="rules"
      ref="ruleForm"
      label-width="100px"
      class="demo-ruleForm"
    >
      <el-form-item label="上级分类" prop="pid">
        <el-select v-model="ruleForm.pid" placeholder="请选择">
          <el-option label="顶级菜单" :value="0"></el-option>
          <el-option
            :label="item.catename"
            :value="item.id"
            v-for="item in menuList"
            :key="item.id"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="分类名称" prop="catename">
        <el-input v-model="ruleForm.catename"></el-input>
      </el-form-item>

      <el-form-item label="图片" prop="img">
        <el-upload
          action="#"
          list-type="picture-card"
          :auto-upload="false"
          :on-change="change"
          :file-list="arr"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
      </el-form-item>

      <el-form-item label="状态" prop="status">
        <el-switch v-model="ruleForm.status"></el-switch>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">{{
          buttontitle
        }}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      arr:[],
      buttontitle: "", //底部名称
      toptitle: "", //面包屑名称
      id: "", //存储要修改的id
      menuList: [], //储存菜单列表的数据
      ruleForm: {
        pid: "",
        catename: "", 
        img: "",
        status: false, //状态  1正常  2禁用  number类型
      },
      rules: {
        title: [{ required: true, message: "请输入菜单名称", trigger: "blur" }],
        pid: [{ required: true, message: "请选择上级菜单", trigger: "change" }],
      },
    };
  },
  mounted() {
    // console.log(this)
    //调用获取菜单列表的方法
    this.getMenuList();
    //获取传过来的id
    this.id = this.$route.query.id;

    if (this.id) {
      //如果有id,说明是通过编辑过来的
      this.toptitle = "商品分类编辑";
      this.buttontitle = "修改";

      //并且调用接口，做到回显功能
      this.$http.get("/cateinfo", { id: this.id }).then((res) => {
        let { status } = res.data.list;
        this.ruleForm = {
          ...res.data.list,
          status: status == 1 ? true : false,
        };
        if(res.data.list.img){
          this.arr.push({
            url:'http://localhost:3000' + res.data.list.img
          })
        }
      });
    } else {
      //反之，说明是通过添加过来的
      this.toptitle = "商品分类添加";
      this.buttontitle = "添加";
    }
  },
  methods: {
    change(file){
      // console.log(file);
      // console.log(fileList);
      this.ruleForm.img = file.raw;
    },
    //定义获取菜单列表的方法
    getMenuList() {
      this.$http.get("/catelist", { pid:0 }).then((res) => {
        // console.log(res)
        this.menuList = res.data.list;
      });
    },

    //添加事件中，传过来的是一个ruleForm字符串，所以this.$refs[formName]这里用中括号
    submitForm(formName) {
      // validate验证必传的项正不正确
      this.$refs[formName].validate((valid) => {
        //valid提交成功，
        if (valid) {
          // alert("submit!");
          this.ruleForm.status = this.ruleForm.status ? 1 : 2;

          let form = new FormData();

          for(let key in this.ruleForm){
            form.append(key, this.ruleForm[key])
          }

          if (this.id) {
            form.append('id', this.id);
            this.$http
              .post("/cateedit", form)
              .then((res) => {
                // console.log(res)
                if (res.data.code == 200) {
                  this.$message({
                    message: "修改成功",
                    type: "success",
                  });
                  setTimeout(() => {
                    this.$router.push("/home/category");
                  }, 500);
                }
              });
          } else {
            //调用菜单添加接口
            this.$http.post("/cateadd", form).then((res) => {
              //如果添加成功，就跳转到菜单列表页
              if (res.data.code == 200) {
                this.$message({
                  message: "添加成功",
                  type: "success",
                });
                setTimeout(() => {
                  this.$router.back();
                }, 500);
              }
              // console.log(res)
            });
          }
        } else {
          //否则失败
          // console.log("error submit!!");
          return false;
        }
      });
    },
  },
};
</script>

<style scoped>
@import "../../style/Menu.css";
</style>