<template>
    <div class="container">
        <div class="table">
            <el-tabs type="card" >
                <el-tab-pane label="未完成" name="first"></el-tab-pane>
                <el-tab-pane label="進行中" name="second"></el-tab-pane>
                <el-tab-pane label="已完成" name="third"></el-tab-pane>
                <el-input v-model="todo"></el-input>
                <button @click="submitHandler">adding</button>

            </el-tabs>

        </div>
        <div class="menu">

            <el-table
                    :data="todos"
                    style="width: 100%"
            >
                <el-table-column type="selection"
                >
                </el-table-column>

                <el-table-column
                    prop="id"
                    label="id"
                    width="40px"
                    show-overflow-tooltip>
                </el-table-column>

                <el-table-column
                        prop="name"
                        label="name"
                        show-overflow-tooltip>
                </el-table-column>


            </el-table>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                todos:[{
                    id:1,
                    name:"",
                    status:0
                }],
                todo:'',
                nextID:2,
                checked:true
            }
        },
        methods:{
            submitHandler(){
                if(this.todo){
                    this.todos.push({
                        id:this.nextID ++,
                        name:this.todo,
                        status:0
                    })
                }
                this.todo=''
            }
        }


    }
</script>

<style>
    .container{
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
    .menu{
        margin-top: 30px;
    }
</style>