<template>
  <div class="page">
    <div class="search">
      <el-input placeholder="专场名称/发布者" v-model="name" style="width:200px;margin-right:20px;"></el-input>
      <el-date-picker v-model="startTime" type="datetime" clearable value-format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期">
      </el-date-picker>
      <span style="display: inline-block;padding: 10px;">至</span>
      <el-date-picker v-model="endTime" type="datetime" clearable value-format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期">
      </el-date-picker> 
      <el-button type="primary" plain icon="el-icon-search" @click="search" style="margin-left:20px;"></el-button>
      <el-button type="primary" @click="handleAdd()">新建专题</el-button>
      <el-button type="danger" @click="batchDel">批量删除</el-button>
    </div>

    <el-table :data="tableData" style="width: 100%;" @selection-change="handleSelectionChange" :default-sort="{prop: 'publicer', order: 'descending'}">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column type="index" label="序号"></el-table-column>
      <el-table-column prop="content" label="标题"></el-table-column>
      <el-table-column prop="nickname" label="发布者" sortable></el-table-column>
      <el-table-column prop="gmtCreate" label="发布时间" sortable></el-table-column>
      <el-table-column label="操作" width="350px">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" plain @click="handleDetail(scope.$index, scope.row)">查看</el-button>
          <el-button size="mini" type="primary" @click="handledialogue(scope.$index, scope.row)">进入问答</el-button>
          <el-button size="mini" type="primary" plain @click="handleEdit(scope.$index, scope.row)">修改</el-button>
          <el-button size="mini" type="danger" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog :visible.sync="dialogVisible" width="30%" center :show-close="flase">
      <el-select placeholder="请选择" style="width:100%;">
        <el-option label="含有违禁词语"></el-option>
        <el-option label="含有铭感词汇"></el-option>
      </el-select>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    
    <el-dialog :visible.sync="dialogDetailVisible" width="50%" center>
      <div class="detail" >
        <div class="up">
          <img :src="serverSrc+detail.coverImage"/>
          <div class="tip">{{detail.title}}</div>
          <div class="name">
            {{detail.presentorName}}
            <span>|</span>{{detail.presentorJob}}</div>
          <div class="texts" v-html="detail.introduction">
            
          </div>
        </div>
        <div class="comment" v-if="comments.length > 0" >
          <div class="title">评论</div>
          <div class="item" v-for="item in comments" :key="item.index">
            <div class="head">
              <div>{{item.nn}}
                <span>{{item.time}}</span>
              </div>
              <div class="del" ref="del" @click="del(item)">删除</div>
            </div>
            <div class="text">
              {{item.title}}
            </div>
            <el-collapse accordion v-model="item.ans" v-if="item.commentVos != undefined" >
              <el-collapse-item name="1">
                 <template slot="title" >
                  回复（{{item.commentVos.length}}）
                </template>
                <div v-for="(items, index) in item.commentVos" :key="index">
                  <div class="head">
                    <div>{{items.nn}}
                      <span>{{items.time}}</span>
                    </div>
                    <div class="del" ref="del" @click="delTwo(items)">删除</div>
                  </div>
                  <div class="text">
                    {{items.content}}
                  </div>
                </div>
              </el-collapse-item>
            </el-collapse>
            
          </div>
        </div>
      </div>
             <!-- 评论分页 -->
        <div class="block"  v-if="comments.length > 0" >
          <el-pagination @current-change="handleCurrentChange"  @size-change="handleSizeChange" 
          :page-sizes="[5,10,15,20,25]"
          :current-page.sync="pn1" :page-size="ps1" layout="total, sizes, prev, pager, next"
          :total="ct">
          </el-pagination>
      </div>
    </el-dialog>
    <el-dialog :title="flag==1?'新增专题':'修改专题'"  :visible.sync="addAndEditVisible" width="50%" center :show-close="false" :close-on-click-modal="false">
      <el-form ref="form" label-width="100px" :model="form" :rules="rules">
        <el-form-item label="标题" prop="title">
          <el-input v-model="form.title"></el-input>
        </el-form-item>
        <el-form-item label="发布者" prop="presentorName">
          <el-input v-model="form.presentorName"></el-input>
        </el-form-item>
        <el-form-item label="职位介绍" prop="presentorJob">
          <el-input v-model="form.presentorJob"></el-input>
        </el-form-item>
        <el-form-item label="回答专家" prop="expertId">
          <el-select placeholder="请选择" v-model="form.expertId">
            <el-option v-for="item in expertlist" :key="item.id" :label="item.telephone" :value="item.id"></el-option>
          </el-select>
        </el-form-item>

          
        <el-form-item label="图片">
          <el-upload 
          class="avatar-uploader"
          action="/api/admin/ask/uploadFile.do" 
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :headers="myHeaders" 
          :before-upload="beforeAvatarUpload">
          <img v-if="imageUrl" :src="flag == 1 ?imageUrl:serverSrc+imageUrl" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
         
        <div class="colums">
          <el-form-item label="开始时间" prop="beginTime">
              <el-date-picker :picker-options="pickerOptions" type="datetime" value-format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期" v-model="form.beginTime" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
            <el-form-item label="结束时间" prop="endTime">
              <el-date-picker type="datetime" :picker-options="pickerOptions"  value-format="yyyy-MM-dd HH:mm:ss"  placeholder="选择日期" v-model="form.endTime" style="width: 100%;">
              </el-date-picker>
            </el-form-item>
        </div>
        <el-form-item label="文章" prop="introduction">
          <!-- <el-input type="textarea" v-model="form.introduction" rows="6"></el-input> -->
           <quill-editor v-model="form.introduction"
                    ref="myQuillEditor"
                    class="editer"
                    :options="editorOption">
                </quill-editor>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancel('form')">取 消</el-button>
        <el-button type="primary" @click="save('form')" :loading="loading">发 布</el-button>
      </span>
    </el-dialog>
   <Pagination :pageCurrent="pn" v-on:refreshCurrentChange="refreshCurrentChange" v-on:refreshSizeChange="refreshSizeChange" :count="count">
    </Pagination>
  </div>
</template>
<script>
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { quillEditor } from 'vue-quill-editor' //调用编辑器
import {quillRedefine} from 'vue-quill-editor-upload'
import Pagination from "@/components/element/Pagination";
import API from "@/components/common/Api";
export default {
  components: { Pagination,quillEditor,quillRedefine},
  data() {    
    
   var checkendTime = (rule,value,callback) => {
      var flag = this.$moment(value).isAfter(this.form.beginTime);
      if(flag){
        callback();
      }else{
        callback(new Error('结束日期不能小于开始日期'));
      }
    };
    
    
    return {
      serverSrc:this.GLOBAL.serverSrc,
      myHeaders: {
        "XIANGYU-ACCESS-TOKEN": sessionStorage.token
      },
      
      pickerOptions: {
          disabledDate: (time) => {
            return time.getTime() < Date.now() - 8.64e7;
          }
        },
      pn: 1,
      ps: 10,
      count: 0,

      pn1: 1,
      ps1: 10,
      ct: 0,
      activeNames:'0',
      flag: '',
      startTime: "",
      endTime: "",
      name: "",
      id: "",
      multipleSelection: [],
      comments:'',
      expertlist:'',
      // 新建专题参数
     
      form: {
        id: "",
        title: "",
        presentorName: "",
        presentorJob: "",
        beginTime: "",
        endTime: "",
        introduction: "",
        coverImage: "",
        expertId:' '
      },

      rules: {
        title: [
          { required: true, message: "请输入标题", trigger: "blur" },
          { min: 1, max: 30, message: "长度在 1 ~ 30 个字符", trigger: "blur" }
        ],
        presentorName: [
          { required: true, message: "请输入发布者名称", trigger: "blur" },
          { min: 1, max: 30, message: "长度在 1 ~ 30 个字符", trigger: "blur" }
        ],
        presentorJob: [
          { required: true, message: "请输入职位名称", trigger: "blur" }
        ],
        introduction: [
          { required: true, message: "请输入文章", trigger: "blur" }
        ],
        beginTime: [
          { required: true, message: "请选择开始时间", trigger: "change" }
        ],
        endTime: [
          { required: true, message: "请选择结束时间", trigger: "change" },
          {validator:checkendTime,trigger:'change'}
        ],
        expertId: [
          { required: true, message: "请选择专家", trigger: "change" }
        ]
      },
      addAndEditVisible: false,
      dialogVisible: false,
      dialogDetailVisible: false,
      tableData: [],
      detail: "",
    
      loading:false,
      imageUrl:'',
      isUpload:false,
      editorOption: {
          modules: {
            toolbar: [
              ['bold', 'italic', 'underline', 'strike'],
              ['blockquote', 'code-block'],
              [{ 'header': 1 }, { 'header': 2 }],
              [{ 'list': 'ordered' }, { 'list': 'bullet' }],
              [{ 'script': 'sub' }, { 'script': 'super' }],
              [{ 'indent': '-1' }, { 'indent': '+1' }],
              [{ 'direction': 'rtl' }],
              [{ 'size': ['small', false, 'large', 'huge'] }],
              [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
              [{ 'font': [] }],
              [{ 'color': [] }, { 'background': [] }],
              [{ 'align': [] }],
              ['clean'],
              ['link', 'image']
            ],
            syntax: {
              highlight: text => hljs.highlightAuto(text).value
            }
          },
          
        },
    };
  },

  methods: {
    
     changeTime(date){
          this.form.startTime = '';
          let beginDateVal = new Date(date).getTime();
          this.pickerBeginDateAfter={
            disabledDate(time) {  //开始时间-结束时间 
              return (time.getTime() < new Date(date).getTime());
            }
          }
      },
    clear(){
        this.form.id = '';
        this.form.title = '';
        this.form.presentorName = '';
        this.form.presentorJob = '';
        this.form.beginTime = '';
        this.form.endTime = '';
        this.form.introduction = '';
        this.form.coverImage = '';
        this.form.nickname = '';
        this.form.expertId = '';
        this.imageUrl = '';
    },
    handleAvatarSuccess(res, file) {
      console.log(file)
        this.imageUrl = URL.createObjectURL(file.raw);
        
        this.form.coverImage = res.data;
        this.isUpload = true;
        if(res.code == 0){
            this.$message({
                message: res.msg,
                type: 'success'
            });
            
        }else{
            this.$message({
                message: res.msg,
                type: 'warning'
            });
        }
      },
      beforeAvatarUpload(file) {
        var  type = true;
          if(file.type != 'image/jpeg' && file.type != 'image/png'){
            type = false;
          }else{
            type = true;
          }
          const isLt2M = file.size / 1024 / 1024 < 2;

          if (!type) {
            this.$message.error('上传图片只能是 JPG或者PNG 格式!');
          }
          if (!isLt2M) {
            this.$message.error('上传图片大小不能超过 2MB!');
          }          
    
          return type && isLt2M;
      },
 
    search() {
      this.pn = 1;
        this.getList();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },

    refreshCurrentChange(val) {
      this.pn = val;
      this.getList();
    },
    refreshSizeChange(val) {
      this.pn = 1;
      this.ps = val;
      this.getList();
    },

    handleCurrentChange(val){
      this.pn1 = val;
      this.lookComment();
    },
    handleSizeChange(val){
      this.ps1 = val;
      this.lookComment();
    },
    // 新建专题
    handleAdd() {
      this.flag = 1;
      this.addAndEditVisible = true;
    },

    // 批量删除
    batchDel(index, row) {
      if (this.multipleSelection.length == 0) {
        this.$message({
          message: "请先选择专题",
          type: "warning"
        });
        return;
      }
      this.$confirm("确定进行此操作？", "提示", {
        type: "warning",
        center: true
      })
        .then(_ => {
          var list = [];
          for (var i in this.multipleSelection) {
            list.push(this.multipleSelection[i].id);
          }
          API.batchDelete(list).then(result => {
            this.getList();
          });
        })
        .catch(_ => {});
    },

    // 查看
    handleDetail(index, row) {
      this.dialogDetailVisible = true;
      this.id = row.id;
      let data = {
        id: row.id
      };
      API.detailLook(data).then(result => {
        console.log(result)
        this.detail = result.data.data;
      });
      this.lookComment();
    },

    // 查看里面的评论
    lookComment() {
      let data = {
        id: this.id,
        pn: this.pn1,
        ps: this.ps1
      };
      API.commentSpecial(data).then(result => {
        this.comments = result.data.data.list;
        this.ct = result.data.data.count;
       
      });
    },

    // 查看里面的评论删除方法
    del(item) {
      this.$confirm("此操作将永久删除, 是否继续？", "提示", {
        type: "warning",
        center: true
      })
        .then(_ => {
          let data = {
            id: item.id
          };
          API.commentDelete(data).then(result => {
            this.lookComment();
          });
        })
        .catch(_ => {});
    },
    delTwo(item){
       this.$confirm("此操作将永久删除, 是否继续？", "提示", {
        type: "warning",
        center: true
      }).then(_ => {
        let data = {
          id:item.id
        };
        API.deleteTwoComment(data).then(result => {
          this.lookComment();
        });
      }).catch(_ => {});
    },
    // 进入问答
    handledialogue(index, row) {
      this.$router.push({ path: "/dialogue", query: { id: row.id } });
    },

    // 删除
    handleDel(index, row) {
      let data = {
        id: row.id
      };
      this.$confirm("此操作将永久删除, 是否继续？", "提示", {
        type: "warning",
        center: true
      }).then(_ => {
          API.deleteProject(data).then(result => {
            this.getList();
          });
      }).catch(_ => {});
    },

   

   
    // 修改
    handleEdit(index, row) {
      this.flag = 2;
      this.addAndEditVisible = true;
      let data = {
        id: row.id
      };
      API.upSpecialInfo(data).then(result => {
        this.modification = result.data.data;
        this.form.title = this.modification.title;
        this.form.presentorName = this.modification.presentorName;
        this.form.introduction = this.modification.introduction;
        this.form.beginTime = this.modification.gmtCreate;
        this.form.endTime = this.modification.endTime;
        this.form.presentorJob = this.modification.presentorJob;
        this.form.coverImage = this.modification.coverImage;
        this.form.id = this.modification.id;
        this.form.expertId = this.modification.expertId;
        this.imageUrl = this.modification.coverImage;
      });
    },
    getUrl(path){
      var list = path.split('=');
      return list[1];
    },
    save(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
            if(this.form.coverImage == '' && this.imageUrl == ''){
              this.$message({
                message: '请上传封面图片',
                type: 'warning'
                });
              return;
            }
            this.loading = true;
            if(this.form.coverImage.indexOf('=') >= 0){
              this.form.coverImage = this.getUrl(this.form.coverImage);
            }
            

            API.Special(this.form).then(result => {
              this.loading = false;
             if(result.data.code != 0){
                this.addAndEditVisible = true;
              }else{
                this.addAndEditVisible = false;
                this.$refs[formName].resetFields();
                this.getList();
              }
              
            });
        } else {
          this.$message({ message: "还有验证未通过的信息", type: "warning" });
          return false;
        }
      });
    },

    cancel(formName) {
      this.$refs[formName].resetFields();
      this.addAndEditVisible = false;
      this.clear();
    },
    getExpert(){
      API.getExpert().then(result => {
        this.expertlist = result.data.data;
      });
    },
    getList() {
      let data = {
        pn: this.pn,
        ps: this.ps,
        name: this.name,
        startTime: this.startTime,
        endTime: this.endTime
      };
      API.getSpecial(data).then(result => {
        this.tableData = result.data.data.list;
        this.count = result.data.data.count;
        this.isUpload = false;
      });
    }
  },

  created() {
    this.getList();
    this.getExpert();
    this.editorOption = quillRedefine(
        {
          // 图片上传的设置
          uploadConfig: {
            action: '/api/base/uploadfile.do',  // 必填参数 图片上传地址
            // 必选参数  res是一个函数，函数接收的response为上传成功时服务器返回的数据
            // 你必须把返回的数据中所包含的图片地址 return 回去
            res: (respnse) => {
                // console.info(respnse.url);
              console.log(response)
              return respnse.url
            },
            methods: 'POST',  // 可选参数 图片上传方式  默认为post
            token: sessionStorage.token,  // 可选参数 如果需要token验证，假设你的token有存放在sessionStorage
            name: 'file',  // 可选参数 文件的参数名 默认为img
            size: 500,  // 可选参数   图片限制大小，单位为Kb, 1M = 1024Kb
            accept: 'image/png, image/jpeg, image/jpg',  // 可选参数 可上传的图片格式
            // start: function (){}
            start: () => {
            },  // 可选参数 接收一个函数 开始上传数据时会触发
            end: () => {
            },  // 可选参数 接收一个函数 上传数据完成（成功或者失败）时会触发
            success: () => {
            },  // 可选参数 接收一个函数 上传数据成功时会触发
            error: () => {
            }  // 可选参数 接收一个函数 上传数据中断时会触发
          },
          // 以下所有设置都和vue-quill-editor本身所对应
          placeholder: '',  // 可选参数 富文本框内的提示语
          theme: '',  // 可选参数 富文本编辑器的风格
          toolOptions: this.editorOption.toolbarOptions,  // 可选参数  选择工具栏的需要哪些功能  默认是全部
          handlers: {}  // 可选参数 重定义的事件，比如link等事件
        }
      )
  }
};
</script>
<style scoped>
.detail .groupImg {
  display: flex;
  flex-wrap: wrap;
  /* margin-top: 5px; */
}
.detail .groupImg img {
  margin-right: 15px;
  margin-top: 15px;
  width: 30%;
  height: 100px;
}
.block {
  text-align: right;
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
.el-col-2 {
  text-align: center;
}
.pagination {
  position: fixed;
  bottom: 30px;
  text-align: right;
  right: 50px;
  width: 100%;
}
.search .label {
  display: inline-block;
  width: 100px;
  padding: 10px 10px 0 0;
}
.search_select {
  display: flex;
  align-items: center;
}
.detail {
  position: relative;
}
.detail .up {
  padding: 10px;
  border-bottom: 1px dashed #e6e6e6;
}
.bottom {
  position: absolute !important;
  width: 100% !important;
  padding: 0 !important;
  left: 0 !important;
  /* display: none; */
}
.up img {
  width: 100%;
  height: 150px;
}
.up .texts {
    width: 100%;
    overflow: hidden;
}
.up .tip {
  font-size: 15px;
  color: #000;
}
.up .name {
}
.up .name span {
  padding: 0 10px;
}
.votes {
  margin-top: 20px;
}
.vote .item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-left: 15px;
}
.vote .item .progress {
  width: 85%;
}
.vote .item .name {
  margin-right: 10px;
  width: 60px;
  text-align: left;
  font-size: 14px;
  color: #333;
}
.vote .item .number {
  color: #409eff;
}
.total {
  text-align: left;
}
.comment {
  padding: 10px;
}
.comment .title {
  font-size: 16px;
  color: #000;
}
.comment .item {
  border-bottom: 1px solid #e6e6e6;
  padding: 10px 0;
}
.comment .item .head {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.comment .item .head div:first-child span {
  padding-left: 10px;
}
.comment .item .del {
  color: #e51c23;
  cursor: pointer;
}
</style>
  