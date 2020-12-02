<template>
    <div>
        <div>
            <el-input size="small" v-model="jl.name" style="width: 300px;" prefix-icon="el-icon-plus"
                      placeholder="add position..."></el-input>
            <el-select v-model="jl.titleLevel" placeholder="titleLevel" size="small"
                       style="margin-left: 5px;margin-right: 5px">
                <el-option
                        v-for="item in titleLevels"
                        :key="item"
                        :label="item"
                        :value="item">
                </el-option>
            </el-select>
            <el-button icon="el-icon-plus" type="primary" size="small" @click="addJobLevel">add</el-button>
        </div>
        <div style="margin-top: 10px">
            <el-table
                    :data="jls"
                    border
                    v-loading="loading"
                    element-loading-text="loading..."
                    element-loading-spinner="el-icon-loading"
                    element-loading-background="rgba(0, 0, 0, 0.8)"
                    size="small"
                    @selection-change="handleSelectionChange"
                    style="width: 80%">
                <el-table-column
                        type="selection"
                        width="55">
                </el-table-column>
                <el-table-column
                        prop="id"
                        label="id"
                        width="55">
                </el-table-column>
                <el-table-column
                        prop="name"
                        label="name"
                        width="150">
                </el-table-column>
                <el-table-column
                        prop="titleLevel"
                        label="titleLevel">
                </el-table-column>
                <el-table-column
                        prop="createDate"
                        label="createDate">
                </el-table-column>
                <el-table-column
                        label="is opened">
                    <template slot-scope="scope">
                        <el-tag type="success" v-if="scope.row.enabled">opened</el-tag>
                        <el-tag type="danger" v-else>not opened</el-tag>
                    </template>
                </el-table-column>
                <el-table-column
                        label="操作">
                    <template slot-scope="scope">
                        <el-button size="small" @click="showEditView(scope.row)">edit</el-button>
                        <el-button size="small" type="danger" @click="deleteHandler(scope.row)">delete</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-button type="danger" size="small" style="margin-top: 10px" :disabled="multipleSelection.length==0"
                       @click="deleteMany">batch delete
            </el-button>

        </div>
        <el-dialog
                title="change position"
                :visible.sync="dialogVisible"
                width="30%">
            <div>
                <table>
                    <tr>
                        <td>
                            <el-tag>title name</el-tag>
                        </td>
                        <td>
                            <el-input size="small" v-model="updateJl.name"></el-input>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <el-tag>title level</el-tag>
                        </td>
                        <td>
                            <el-select v-model="updateJl.titleLevel" placeholder="title level" size="small"
                                       style="margin-left: 5px;margin-right: 5px">
                                <el-option
                                        v-for="item in titleLevels"
                                        :key="item"
                                        :label="item"
                                        :value="item">
                                </el-option>
                            </el-select>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <el-tag>is opened</el-tag>
                        </td>
                        <td>
                            <el-switch
                                    v-model="updateJl.enabled"
                                    active-text="open"
                                    inactive-text="close">
                            </el-switch>
                        </td>
                    </tr>
                </table>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button size="small" @click="dialogVisible = false">cancel</el-button>
    <el-button size="small" type="primary" @click="doUpdate">confirm</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "JobLevelMana",
        data() {
            return {
                dialogVisible: false,
                loading: false,
                multipleSelection: [],
                updateJl: {
                    name: '',
                    titleLevel: '',
                    enabled: false
                },
                jl: {
                    name: '',
                    titleLevel: ''
                },
                jls: [],
                titleLevels: [
                    'senior level',
                    'deputy senior level',
                    'junior level',
                    'primary level',
                    'member level',
                ]
            }
        },
        mounted() {
            this.initJls();
        },
        methods: {
            deleteMany() {
                this.$confirm('This operation will delete【' + this.multipleSelection.length + '】records, do you still continue?', 'Warning', {
                    confirmButtonText: 'confirm',
                    cancelButtonText: 'cancel',
                    type: 'warning'
                }).then(() => {
                    let ids = '?';
                    this.multipleSelection.forEach(item => {
                        ids += 'ids=' + item.id + '&';
                    })
                    this.deleteRequest("/system/basic/joblevel/" + ids).then(resp => {
                        if (resp) {
                            this.initJls();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: 'has cancel delete operaion'
                    });
                });
            },
            doUpdate() {
                this.putRequest("/system/basic/joblevel/", this.updateJl).then(resp => {
                    if (resp) {
                        this.initJls();
                        this.dialogVisible = false;
                    }
                })
            },
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            showEditView(data) {
                Object.assign(this.updateJl, data);
                this.dialogVisible = true;
            },
            deleteHandler(data) {
                this.$confirm('This operation will delete【' + data.name + '】jobTitle, do you still continue?', 'Warning', {
                    confirmButtonText: 'confirm',
                    cancelButtonText: 'cancel',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/joblevel/" + data.id).then(resp => {
                        if (resp) {
                            this.initJls();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: 'has cancel delete operation'
                    });
                });
            },
            addJobLevel() {
                if (this.jl.name && this.jl.titleLevel) {

                    this.postRequest("/system/basic/joblevel/", this.jl).then(resp => {
                        if (resp) {
                            this.initJls();
                        }
                    });
                } else {
                    this.$message.error("can not be empty!");
                }
            },
            initJls() {
                this.loading = true;
                this.getRequest("/system/basic/joblevel/").then(resp => {
                    this.loading = false;
                    if (resp) {
                        this.jls = resp;
                        this.jl = {
                            name: '',
                            titleLevel: ''
                        };
                    }
                })
            }
        }
    }
</script>

<style scoped>

</style>