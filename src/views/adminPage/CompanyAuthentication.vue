<template>
    <el-container>
        <el-aside width="15vw" style="background-color:#444076">
            <img class="logo" alt="logo" src="../../assets/logo.png">
            <el-menu
                class="el-menu"
                default-active="/admin/companyAuthentication"
                router=true
                style="height: 89vh;"
                active-text-color="#FFCAA6"
                background-color="#444076"
                text-color="#fff"
                open="1"
            >
                <el-sub-menu index="1">
                <template #title>
                    <el-icon><User /></el-icon>
                    <span class="menu-item1">审查实名</span>
                </template>
                    <el-menu-item class="menu-item2" index="/admin/personAuthentication">&emsp;个人认证</el-menu-item>
                    <el-menu-item class="menu-item2" index="/admin/companyAuthentication">&emsp;企业认证</el-menu-item>
                </el-sub-menu>
                <el-menu-item index="/admin/order">
                <template #title>
                    <el-icon><Memo /></el-icon>
                    <span class="menu-item1">订单申诉</span>
                </template>
                </el-menu-item>
                <el-sub-menu index="2">
                <template #title>
                    <el-icon><Notification /></el-icon>
                    <span class="menu-item1">兼职管理</span>
                </template>
                    <el-menu-item class="menu-item2" index="/admin/job">&emsp;审核兼职</el-menu-item>
                    <el-menu-item class="menu-item2" index="/admin/jobReport">&emsp;兼职举报</el-menu-item>
                    <el-menu-item class="menu-item2" index="/admin/jobType">&emsp;兼职类型</el-menu-item>
                </el-sub-menu>
                <!-- <el-menu-item index="/admin/notification">
                <template #title>
                    <el-icon><Bell /></el-icon>
                    <span class="menu-item1">系统通知</span>
                </template>
                </el-menu-item> -->
                <el-menu-item index="/admin/notice">
                <template #title>
                    <el-icon><Reading /></el-icon>
                    <span class="menu-item1">公告管理</span>
                </template>
                </el-menu-item>
            </el-menu>  
        </el-aside>
        <el-container class="" style="height: 100vh">
            <el-header class="header">
                <el-row>
                    <el-col :span="22">
                    </el-col>
                    <el-col :span="2">
                        <el-button plain type="primary">
                            <router-link to="/">退出登录</router-link>
                        </el-button>
                    </el-col>
                </el-row>
            </el-header>
            <el-main style="background-color: #F0F1F6;">
                <el-card class="card">
                    <template #header>
                    <div class="card-header">
                        <span class="title">申请列表</span>
                    </div>
                    </template>
                    <el-table
                        :data="applyList.slice((page-1) * limit, page * limit)" 
                        style="width: 100%"
                        size="large"
                        height="430"
                        highlight-current-row
                        @current-change="getCurrentRow"
                    >
                        <el-table-column label="序号" type="index" width="80" />
                        <el-table-column prop="recruiterId" label="招聘方ID" sortable width="120"/>
                        <el-table-column prop="companyType" label="公司类型" width="120"/>
                        <el-table-column prop="companyName" label="公司名称" width="120"/>
                        <el-table-column prop="companyId" label="资质证号"/>
                        <el-table-column label="证件照片">
                            <template #default="scope">
                                <el-image style="width: 12vw; height: 9vw" :src="scope.row.certification" :fit="fit" />
                            </template>
                        </el-table-column>
                        <el-table-column prop="applyTime" sortable label="申请时间"/>
                        <el-table-column
                            prop="checkStatus"
                            label="状态"
                            width="100"
                            :filters="[
                                { text: '已通过', value: '已通过' },
                                { text: '未通过', value: '未通过' },
                                { text: '未审核', value: '未审核' },
                            ]"
                            :filter-method="filterTag"
                            filter-placement="bottom-end"
                            >
                            <template #default="scope">
                                <el-tag
                                    v-if="scope.row.checkStatus == '未审核'"
                                    disable-transitions
                                >未审核</el-tag>
                                <el-tag
                                    v-if="scope.row.checkStatus == '已通过'"
                                    type="success"
                                    disable-transitions
                                >已通过</el-tag>
                                <el-tag
                                    v-if="scope.row.checkStatus =='未通过'"
                                    type="danger"
                                    disable-transitions
                                >未通过</el-tag>
                            </template>
                        </el-table-column>
                        <el-table-column fixed="right" label="操作" width="120">
                            <template #default="operation">
                                <el-button v-if="operation.row.checkStatus =='未审核'" link type="primary" size="small" @click="dialogFormVisible = true"
                                >审核</el-button>
                                <el-button v-if="operation.row.checkStatus !='未审核'" link type="primary" size="small" @click="dialogFormVisible = true"
                                >查看</el-button>
                                <!-- <el-button link type="primary" size="small" @click="confirmDelete">删除</el-button> -->
                            </template>
                        </el-table-column>
                    </el-table>
                    <el-row justify="center">
                        <div class="pagination">
                            <div class="pagination_style">&emsp;</div>
                            <el-pagination
                            :current-page="page"
                            :page-size="limit"
                            background
                            layout="prev, pager, next"
                            :total="total"
                            @current-change="handleCurrentChange"
                            />
                        </div>
                    </el-row>
                    <el-dialog v-model="dialogFormVisible" title="申请详情" align-center draggable>
                        <el-form :model="apply">
                            <el-form-item v-if="apply.checkStatus !='未审核'" label="审核时间" label-width="80px">
                                <el-input disabled v-model="apply.checkTime" autocomplete="off" />
                            </el-form-item>
                            <el-form-item label="公司类型" label-width="80px">
                                <el-tag disable-transitions>{{ apply.companyType }}</el-tag>
                            </el-form-item>
                            <el-form-item label="公司名称" label-width="80px">
                                <el-input disabled v-model="apply.companyName" autocomplete="off" />
                            </el-form-item>
                            <el-form-item label="资质证号" label-width="80px">
                                <el-input disabled v-model="apply.companyId" autocomplete="off" />
                            </el-form-item>
                            <el-form-item label="证件照片" label-width="80px">
                                <el-image style="width: 16vw; height: 12vw" :src="apply.certification" :fit="fit" />
                            </el-form-item>
                            <el-form-item v-if="apply.checkStatus!='未审核'" label="审核状态" label-width="80px">
                                <el-tag
                                    v-if="apply.checkStatus == '已通过'"
                                    type="success"
                                    disable-transitions
                                >已通过</el-tag>
                                <el-tag
                                    v-if="apply.checkStatus =='未通过'"
                                    type="danger"
                                    disable-transitions
                                >未通过</el-tag>
                            </el-form-item>
                            <el-form-item v-if="apply.checkStatus=='未通过'" label="结果反馈" label-width="80px">
                                <el-input disabled v-model="apply.result" autocomplete="off" type="textarea" :rows="3"/>
                            </el-form-item>
                        </el-form>
                        <template #footer>
                            <span class="dialog-footer">
                                <el-button v-if="apply.checkStatus!='未审核'" type="primary" @click="dialogFormVisible=false">返回</el-button>
                                <el-button v-if="apply.checkStatus=='未审核'" type="danger" plain @click="dialogFormVisible1 = true;">驳回</el-button>
                                <el-button v-if="apply.checkStatus=='未审核'" type="primary" plain @click="pass">通过</el-button>
                            </span>
                        </template>
                    </el-dialog>
                    <el-dialog style="width:30%" v-model="dialogFormVisible1" title="结果反馈" align-center draggable>
                        <el-input v-model="result" autocomplete="off" type="textarea" :rows="3"/>
                        <template #footer>
                            <span class="dialog-footer">
                                <el-button type="danger" plain @click="dialogFormVisible1=false;">取消</el-button>
                                <el-button type="primary" plain @click="noPass">确认</el-button>
                            </span>
                        </template>
                    </el-dialog>
                </el-card>
            </el-main>
        </el-container>
    </el-container>

</template>

<script>
import { ElMessage,ElMessageBox } from 'element-plus'
import 'element-plus/es/components/message/style/index'
import 'element-plus/es/components/message-box/style/index'

export default {
    name: 'adminNotice',
    data() {
        return {
            page: 1,
            limit: 8,
            total: 6,
            applyList:[{}],
            dialogFormVisible: false,
            dialogFormVisible1: false,
            result:null,
            apply:{},
            display:false,
            isPass:true,
        }
    },
    methods: {
        handleCurrentChange(val){
            this.page=val
        },
        getCurrentRow(value){
            if(value!=null){
                console.log(value);
                this.apply=value;
                console.log(this.apply);
            }
        },
        noPass(){
            console.log("不通过");
            if(this.result==""||this.result==null){
                ElMessage({
                    message: "反馈不能为空",
                    type: 'error',
                })
                return;
            }
            this.$axios({
                method: 'post',
                url: '/api/admin/auditCompanyAuthentication',
                data:{
                    applyId: this.apply.applyId,
                    checkStatus: '未通过',
                    result: this.result
                }
            })
            .then(res => {
                if(res.data.code==200){
                    ElMessage({
                        message: "已驳回该申请",
                        type: 'success',
                    })
                    this.$router.go(0);
                }
                else{
                    ElMessage({
                        message: "操作失败",
                        type: 'error',
                    })
                }
            })
            .catch(function (error) {
                console.log(error);
                ElMessage({
                    message: "操作失败",
                    type: 'error',
                })
            })
        },
        pass(){
            ElMessageBox.confirm(
                '确认通过该申请?',
                '提示',
                {
                    distinguishCancelAndClose: true,
                    confirmButtonText: '确认',
                    cancelButtonText: '取消',
                }
            )
            .then(() => {
                this.dialogFormVisible = false;
                this.$axios({
                    method: 'post',
                    url: '/api/admin/auditCompanyAuthentication',
                    data:{
                        applyId: this.apply.applyId,
                        checkStatus: '已通过'
                    }
                })
                .then(res => {
                    if(res.data.code==200){
                        ElMessage({
                            message: "已通过该申请",
                            type: 'success',
                        })
                        this.$router.go(0);
                    }
                    else{
                        ElMessage({
                            message: "操作失败",
                            type: 'error',
                        })
                    }
                })
                .catch(function (error) {
                    console.log(error);
                    ElMessage({
                        message: "操作失败",
                        type: 'error',
                    })
                })
            })

        },
        filterTag(value, row) {
            return row.checkStatus===value;
        },
    },
    created(){
        this.$axios({
            method: 'get',
            url: '/api/admin/getCompanyAuthenticationList',
        })
        .then(res => {
            if(res.data.code==200){
                this.applyList=res.data.data.companyauthen_list;
                this.total=this.applyList.length;
                console.log('企业认证申请列表'+this.applyList);
            }
            else{
                this.applyList=null
            }
        })
        .catch(function (error) {
            console.log(error);
        })
    },
}


</script>

<style scoped>
.header{
    width: 100%;
    height: 10vh;
    line-height: 10vh;
    border-bottom:3px solid gainsboro;
}
.header .title{
    color: #444076;
    font-size: large;
    padding-left: 1vw;
}
.logo{
    display: inline-block;
    height: 10vh;
    background-color: #444076;
    width: 90%;
}
.el-menu{
    background-color: #444076;
}
.menu-item1{
    font-size: large;
}
.menu-item2{
    color: #fff;
    font-size: medium;
}
.el-menu-item.is-active{
    background-color: #444076;
    color: #FFCAA6;
}
.card{
    height: 99%;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
}
.card-header .title{
    padding-left: 10px;
    color: #444076;
    border-left: 8px solid #FFCAA6;
    font-size: 20px;
}
.card-header .button{
    height: 30px;
    width: 80px;
}

.pagination{
--el-color-primary: #444076;
display: block;
margin: 0 auto;
}
.el-pagination {
--el-pagination-font-size: 16px;
--el-pagination-button-color: #024886;
--el-pagination-button-disabled-bg-color:#fff;
--el-pagination-button-bg-color:#fff;
--el-pagination-hover-color: #167fdb;
}

.btn{
    color: #FFC353;
    height: 30px;
    width: 80px;
}
.btn .text{
    text-decoration:none;
    color: #023764;
} 
</style>