<template>
    <div>
        <el-table
                ref="multipleTable"
                :data="tableData"
                stripe
                max-height="430"
                border
                :cell-style="{paddingTop:'6px',paddingBottom:'6px'}"
                highlight-current-row
                header-cell-class-name="table-header"
                @current-change="handleCurrentRowChange"
                @selection-change="handleSelectionChange"
                style="width: 100%">
            <template v-if="selectType=='radio'">
                <el-table-column
                        label="选择"
                        width="50">
                    <template slot-scope="scope">
                        <el-radio v-model="singleSelection" :label="scope.row" style="padding-left: 6px;">&nbsp;</el-radio>
                    </template>
                </el-table-column>
            </template>
            <template v-else-if="selectType=='checkbox'">
                <el-table-column
                        label="选择"
                        type="selection"
                        width="50">
                </el-table-column>
            </template>
            <el-table-column
                    type="index"
                    label="序号"
                    width="60">
                <template scope="scope"><span>{{scope.$index+(page.currentPage - 1) * page.pageSize + 1}} </span></template>
            </el-table-column>
            <template v-for="(item,i) in head">
                <template>
                    <el-table-column
                            :prop="item.prop"
                            :label="item.label"
                            :width="item.width?item.width:''"
                            :key="i"
                            :formatter="item.formatter"
                            show-overflow-tooltip>
                    </el-table-column>
                </template>
            </template>
        </el-table>
        <div class="block">
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="currentPage"
                    :page-sizes="[10, 20, 30, 50]"
                    :page-size="pageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total"
                    @prev-click="prevClick"
                    @next-click="nextClick"
            >
            </el-pagination>
        </div>
    </div>
</template>
<script>
    import Vue from 'vue';
    import ElementUI from 'element-ui';
    import 'element-ui/lib/theme-chalk/index.css';
    import axios from 'axios';
    Vue.prototype.axios = axios
    Vue.use(ElementUI);

    export default {
        name: 'LzTable',
        props:{
            url:String,
            selectType:String,
            head:Array
        },
        data: function() {
            return {
                queryParams:{},
                tableData:[],
                currentPage: 1,
                page:{
                    currentPage: 1,
                    pageSize: 10,
                    total: 0
                },
                queryFlag: true, //用于防止同时触发@size-chang @current-change
                currentRow: null,
                singleSelection: null, //table单选
                multipleSelection: [],//多选
                //属性table
                defaultProps: {
                    children: 'children',
                    label: 'label'
                }
            }
        },
        components:{
        },
        created() {

        },
        methods: {
            handleSizeChange(val) {
                this.page.pageSize = val;
                this.page.currentPage = 1;
                this.queryData();
            },
            handleCurrentChange(val) {
                this.page.currentPage = val;
                this.queryData();
            },
            prevClick(){
                this.currentPage = this.currentPage-1;
            },
            nextClick(){
                this.currentPage = this.currentPage+1;
            },
            queryData(){
                this.singleSelection = null;
                var params = this.queryParams;
                params.pageSize = this.page.pageSize;
                params.pageNum = this.page.currentPage;
                axios
                    .get(this.url,{
                        params: params
                    })
                    .then(response => {
                        if(response.data && response.data.success){
                            this.tableData = response.data.list;
                            this.total = response.data.total;
                        }else{
                            this.$message({
                                showClose: true,
                                message: response.data.msg,
                                type: 'error'
                            });
                        }
                        this.queryFlag = true;
                    })
                    .catch(function (error) { // 请求失败处理
                        console.log(error);
                    });
            },
            //行变化
            handleCurrentRowChange(currentRow) {
                this.currentRow = currentRow;
            },
            handleClick(row) {
                console.log(row);
            },
            handleSelectionChange(val) {
                this.multipleSelection = val;
            }
        },
        mounted(){
        }
    }
</script>
<style scoped>

</style>
