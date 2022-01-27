<template>
    <div class="app-container">
        医院设置列表
        <!-- 查询按钮 -->
        <el-form :inline="true" class="demo-form-inline">
            <el-form-item>
                <el-input v-model="searchObj.hosname" placeholder="医院名称"/>
            </el-form-item>
            <el-form-item>
                <el-input v-model="searchObj.hoscode" placeholder="医院编号"/>
            </el-form-item>
            <el-button type="primary" icon="el-icon-search" @click="getList()">查询</el-button>
        </el-form>
        
        <!-- 工具条 -->
        <div>
            <el-button type="danger" size="mini" @click="removeRows()">批量删除</el-button>
        </div>

        <!-- banner列表 -->
        <el-table :data="list" stripe style="width: 100%" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"/>
            <el-table-column type="index" width="50" label="序号"/>
            <el-table-column prop="hosname" label="医院名称"/>
            <el-table-column prop="hoscode" label="医院编号"/>
            <el-table-column prop="apiUrl" label="api基础路径" width="200"/>
            <el-table-column prop="contactsName" label="联系人姓名"/>
            <el-table-column prop="contactsPhone" label="联系人手机"/>
            <el-table-column label="状态" width="80">
            <template slot-scope="scope">
                {{ scope.row.status === 1 ? '可用' : '不可用' }}
            </template>
            </el-table-column>

            <el-table-column label="操作" width="280" align="center">
                <template slot-scope="scope">
                    <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">删除</el-button>
                    <el-button type="primary" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">锁定</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">解锁</el-button>
                </template>
            </el-table-column>

        </el-table>
        
        <!-- 分页 -->
        <el-pagination
            :current-page="current"
            :page-size="limit"
            :total="total"
            style="padding: 30px 0; text-align: center;"
            layout="total, prev, pager, next, jumper"
            @current-change="getList"
        />
    </div>
</template>

<script>
import hospset from '@/api/hospset'

export default {
    // 定义变量和初始值
    data() {
        return {
            current:1,
            limit:3,
            searchObj:{},
            list:[],
            total: 0,
            multipleSelection: []
        }
    },
    created() {
        // 调用methods中的方法在页面渲染前获取数据
        this.getList()
    },
    methods: {
        //医院设置列表的方法
        getList(page=1) {
            this.current = page
            hospset.getHospSetList(this.current, this.limit, this.searchObj)
                .then(response => { // 接口返回的数据
                    this.list = response.data.records
                    this.total = response.data.total
                    console.log(this.list)
                })
                .catch(error => {  // 接口返回的错误信息
                    console.log(error)
                })
        },
        removeDataById(id) {
            this.$confirm('此操作将永久删除该医院设置信息, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                // 调用接口
                hospset.deleteHospSet(id)
                    .then(response =>{
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        // 刷新页面
                        this.getList(1)
                    })
            })
        },

        // 获取复选框的id值
        handleSelectionChange(selection) {
            this.multipleSelection = selection
        },

        // 批量删除方法
        removeRows() {
            this.$confirm('此操作将永久删除该医院设置信息, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var idList = []
                for (var i=0;i<this.multipleSelection.length;i++) {
                    var obj = this.multipleSelection[i]
                    var id = obj.id
                    idList.push(id)
                }
                // 调用接口
                hospset.batchHospSet(idList)
                    .then(response =>{
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        // 刷新页面
                        this.getList(1)
                    })
            })
        }
    }
}
</script>