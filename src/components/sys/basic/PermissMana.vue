<template>
    <div
            v-loading="globalLoading"
            element-loading-text="loading..."
            element-loading-spinner="el-icon-loading"
            element-loading-background="rgba(0, 0, 0, 0.8)"
    >
        <div class="permissManaTool">
            <el-input size="small" placeholder="please input name" v-model="role.name">
                <template slot="prepend">ROLE_</template>
            </el-input>
            <el-input size="small" placeholder="please input roleName" v-model="role.nameZh"
                      @keydown.enter.native="doAddRole"></el-input>
            <el-button type="primary" size="small" icon="el-icon-plus" @click="doAddRole">add role</el-button>
        </div>
        <div class="permissManaMain">
            <el-collapse v-model="activeName"
                         v-loading="loading"
                         element-loading-text="loading..."
                         element-loading-spinner="el-icon-loading"
                         element-loading-background="rgba(0, 0, 0, 0.8)"
                         accordion
                         @change="change">
                <el-collapse-item :title="r.nameZh" :name="r.id" v-for="(r,index) in roles" :key="index">
                    <el-card class="box-card">
                        <div slot="header" class="clearfix">
                            <span>resources available</span>
                            <el-button style="float: right; padding: 3px 0;color: #ff0000;" icon="el-icon-delete"
                                       type="text" @click="deleteRole(r)"></el-button>
                        </div>
                        <div>
                            <el-tree
                                    show-checkbox
                                    node-key="id"
                                    ref="tree"
                                    :key="index"
                                    :default-checked-keys="selectedMenus"
                                    :data="allmenus" :props="defaultProps"></el-tree>
                            <div style="display: flex;justify-content: flex-end">
                                <el-button @click="cancelUpdate">cancel change</el-button>
                                <el-button type="primary" @click="doUpdate(r.id,index)">confirm change</el-button>
                            </div>
                        </div>
                    </el-card>
                </el-collapse-item>
            </el-collapse>
        </div>
    </div>
</template>

<script>
    export default {
        name: "PermissMana",
        data() {
            return {
                role: {
                    name: '',
                    nameZh: ''
                },
                allmenus: [],
                activeName: -1,
                selectedMenus: [],
                roles: [],
                loading: false,
                globalLoading: false,
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },
        mounted() {
            this.initRoles();
        },
        methods: {
            deleteRole(role) {
                this.$confirm('This operation will delete【' + role.nameZh + '】role, do you still continue?', 'Warning', {
                    confirmButtonText: 'confirm',
                    cancelButtonText: 'cancel',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/permiss/role/" + role.id).then(resp => {
                        if (resp) {
                            this.initRoles();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: 'has cancel delete operation'
                    });
                });
            },
            doAddRole() {
                if (this.role.name && this.role.nameZh) {
                    this.globalLoading = true;
                    this.postRequest("/system/basic/permiss/role", this.role).then(resp => {
                    this.globalLoading = false;
                        if (resp) {
                            this.role.name = '';
                            this.role.nameZh = '';
                            this.initRoles();
                        }
                    })
                } else {
                    this.$message.error('can not be empty');
                }
            },
            cancelUpdate() {
                this.activeName = -1;
            },
            doUpdate(rid, index) {
                let tree = this.$refs.tree[index];
                let selectedKeys = tree.getCheckedKeys(true);
                let url = '/system/basic/permiss/?rid=' + rid;
                selectedKeys.forEach(key => {
                    url += '&mids=' + key;
                })
                this.putRequest(url).then(resp => {
                    if (resp) {
                        this.activeName = -1;
                    }
                })
            },
            change(rid) {
                if (rid) {
                    this.initAllMenus();
                    this.initSelectedMenus(rid);
                }
            },
            initSelectedMenus(rid) {
                this.getRequest("/system/basic/permiss/mids/" + rid).then(resp => {
                    if (resp) {
                        this.selectedMenus = resp;
                    }
                })
            },
            initAllMenus() {
                this.getRequest("/system/basic/permiss/menus").then(resp => {
                    if (resp) {
                        this.allmenus = resp;
                    }
                })
            },
            initRoles() {
                this.loading = true;
                this.getRequest("/system/basic/permiss/").then(resp => {
                    this.loading = false;
                    if (resp) {
                        this.roles = resp;
                    }
                })
            }
        }
    }
</script>

<style>
    .permissManaTool {
        display: flex;
        justify-content: flex-start;
    }

    .permissManaTool .el-input {
        width: 300px;
        margin-right: 6px;
    }

    .permissManaMain {
        margin-top: 10px;
        width: 700px;
    }
</style>