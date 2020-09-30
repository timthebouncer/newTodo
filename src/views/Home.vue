<template>
  <div class="container">
    <el-tabs v-model="status" style="width: 100%;">
      <el-tab-pane label="未完成" name="first"></el-tab-pane>
      <el-tab-pane label="進行中" name="second"></el-tab-pane>
      <el-tab-pane label="已完成" name="third"></el-tab-pane>
    </el-tabs>
    <div style="display: flex; justify-content: space-between;width: 100%;">

      <div style="display: flex;">
        <input type="text" class="input" v-model="search" @keydown.enter="searchHandler" >
        <el-button slot="append" icon="el-icon-search" size="small"></el-button>
      </div>

      <el-popover
              placement="bottom-end"
              title="新增待辦事項"
              width="200"
              trigger="click"
              @after-enter="focusAdd"
      >
        <template slot="reference">
          <el-button size="small" type="primary">新增</el-button>
        </template>

        <input type="text" ref="autofocus" class="input" v-model="todo" @blur="cancel" @keydown.enter="submitHandler">
      </el-popover>
    </div>
    <div class="menu" style="width: 100%;">
      <el-table :data="filterStatus" style="width: 100%" @select="complete">
        <el-table-column type="selection"> </el-table-column>

        <el-table-column
          prop=""
          label=""
          width="100px"
          show-overflow-tooltip
        >
        </el-table-column>

        <el-table-column label="名稱">
          <template slot-scope="scope">
            <div v-if="edit !== scope.$index">{{scope.row.name}}</div>
            <div v-else>
              <el-input v-model="scope.row.name" @blur="cancel" ref="autofocus"></el-input>
            </div>
          </template>
        </el-table-column>


          <el-table-column width="200px">
          <template slot-scope="scope">

            <el-button @click="startHandler(scope.$index,scope.row)" size="small" icon="el-icon-caret-right"></el-button>


              <el-button type="primary" size="mini" @click="dialogVisible123(scope.$index,scope.row)">編輯</el-button>

            <el-button
                      @click="deleteHandler(scope.$index,result)" type="primary" size="mini">刪除
            </el-button>

          </template>
          </el-table-column>
         </el-table>

    </div>
          <div class="dialog">
            <el-dialog
                    title="編輯代辦事項"
                    :visible.sync="dialogVisible"
                    width="formLabelWidth"
            >

              <div>

                <div class="edit" >Name: <el-input v-model="editDialog" :disabled="status !== 'first' ? true:false"></el-input></div>
                <div class="edit" >Description: <el-input v-model="editdescription" style="height: 50px" :disabled="status !== 'first' ? true:false"></el-input></div>
<!--                <div class="edit">Description:<input v-model="editdescription" type="text" style="height: 50px" :disabled="status === 'third' ? true:false"></div>-->
              </div>

              <span slot="footer" class="dialog-footer">
                <div v-if="status === 'third'" disabled></div>
                <div v-else>
                    <el-button type="primary" @click="editHandler()">確定</el-button>
                    <el-button @click="dialogVisible = false" >取消</el-button>
                </div>

              </span>

            </el-dialog>
          </div>

  </div>
</template>

<script>
    import axios from 'axios'
export default {
  data() {
    return {
      status:"first",
      todo: "",
      nextID: 1,
      checked: true,
      edit:null,
      search:'',
      match:'',
      result:[],
      editDialog:'',
      editdescription:'',
      dialogVisible: false,
      input:"",
      statusA:''
    };
  },
    created() {
        axios.get('http://localhost:3000/todos')
        .then((res)=>{
            this.result = res.data
        }).catch((err)=>{
            console.log(err)
        })
      },
    computed:{
    filterStatus(){
        // console.log(this.result)
      return this.result.filter(item=>{
        return  item.status === this.status && item.name.includes(this.match)
      })
    }
  },
  methods: {
    submitHandler() {
        let name = this.todo
        if(name) {
          axios.post('http://localhost:3000/todos',{
              name:this.todo,status:"first"
          })
          .then(()=>{
              axios.get('http://localhost:3000/todos')
              .then((res)=>{
                  this.result = res.data
              })
          }).catch((err)=>{
              console.log(err)
          })
      }
      this.todo = "";
      this.$nextTick(() => {
        this.$refs.autofocus.focus()
      })
    },
    deleteHandler(index){
        let target = this.result[index]
        if(this.todo !== null){
          if(confirm(`確定刪除?`)){
            axios.delete('http://localhost:3000/todos/' + target.id)
            .then(()=>{
                  axios.get('http://localhost:3000/todos')
                  .then((res)=>{
                      this.result.splice(index,1)
                    this.result = res.data
                  })
            }).catch((err)=>{
              console.log(err)
            })
            
          }
        }
    },
      dialogVisible123(index,row){
          this.dialogVisible = true
          axios.get('http://localhost:3000/todos/'+ row.id)
              .then((res)=>{
                 this.editDialog = res.data.name
                  this.editdescription = res.data.description
                this.input = res.data.id
                this.statusA = res.data.status
              }).catch((err)=>{
              console.log(err)
          })
      },
    editHandler(){
      this.dialogVisible = false
      if(this.editDialog || this.editdescription){
          axios.put('http://localhost:3000/todos/'+ this.input,{
              name: this.editDialog,
              description:this.editdescription,
              status:this.statusA
          })
          .then(()=>{
            axios.get('http://localhost:3000/todos/')
            .then((res)=>{
              console.log(res)
              this.result = res.data
            })
          }).catch((err)=>{
              console.log(err)
          })

      }
      this.$nextTick(() => {
        this.$refs.autofocus.focus()
      })

    },
    cancel(){
      this.edit = null
      this.todo = null
      if(this.editDialog != null){
        return this.editDialog = null
      }
      this.editdescription = null
    },
    complete(selection,row){
      // console.log(row.id)
      if(row.status === "first" || row.status === "second"){
          axios.put('http://localhost:3000/todos/'+ row.id,{
              name:row.name,
              description:row.description || "",
              status:"third"
          })
          .then((res)=>{
              row.status = res.data.status
          })
      }
    },
    startHandler(selection,row){
      row.status = "second"
    },
    focusAdd(){
      this.$nextTick(() => {
        this.$refs.autofocus.focus()
      })
    },
    searchHandler(){
      console.log(1)
        let pattern = new RegExp("^[\u4e00-\u9fa5_A-z0-9]+$")
        if(pattern.test(this.search))
        {this.match = this.search
           }
        else {
            return this.match = ""
        }
    }
  },
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 768px;
  margin: 0 auto;
}
.menu {
  margin-top: 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04)
}
  .input{
    -webkit-appearance: none;
    background-color: #FFF;
    background-image: none;
    border-radius: 4px;
    border: 1px solid #DCDFE6;
    box-sizing: border-box;
    color: #606266;
    display: inline-block;
    font-size: inherit;
    height: 32px;
    line-height: 32px;
    outline: 0;
    padding: 0 15px;
    transition: border-color .2s cubic-bezier(.645,.045,.355,1);
    width: 100%;
  }
  .input:focus {
    border-color: #409EFF;
    outline: 0;
  }

  /*.el-dialog{*/
  /*  height: 250px;*/
  /*  display: flex;*/
  /*  flex-direction: column;*/
  /*  align-content: space-around;*/
  /*}*/
  .edit{
    display: flex;
    /*-webkit-appearance: none;*/
    background-color: #FFF;
    color: #606266;
    /*display: inline-blo/ck;*/
    font-size: inherit;
    height: 40px;
    line-height: 32px;
    padding: 0 15px;
  }
</style>