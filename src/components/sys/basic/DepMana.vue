<template>
    <div style="width: 500px;">
        <el-input
                placeholder="Please input department to search..."
                prefix-icon="el-icon-search"
                v-model="filterText">
        </el-input>

        <el-tree
                :data="deps"
                :props="defaultProps"
                :expand-on-click-node="false"
                :filter-node-method="filterNode"
                ref="tree">
            <span class="custom-tree-node" style="display: flex;justify-content: space-between;width: 100%;"
                  slot-scope="{ node, data }">
        <span>{{data.name }}</span>
        <span>
          <el-button
                  type="primary"
                  size="mini"
                  class="depBtn"
                  @click="() => showAddDepView(data)">
            add
          </el-button>
          <el-button
                  type="danger"
                  size="mini"
                  class="depBtn"
                  @click="() => deleteDep(data)">
            delete
          </el-button>
        </span>
      </span>
        </el-tree>
        <el-dialog
                title="add department"
                :visible.sync="dialogVisible"
                width="50%">
            <div>
                <table>
                    <tr>
                        <td>
                            <el-tag>superior department</el-tag>
                        </td>
                        <td>{{pname}}</td>
                    </tr>
                    <tr>
                        <td>
                            <el-tag>department name</el-tag>
                        </td>
                        <td>
                            <el-input v-model="dep.name" placeholder="department name..."></el-input>
                        </td>
                    </tr>
                </table>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">cancel</el-button>
    <el-button type="primary" @click="doAddDep">confirm</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "DepMana",
        data() {
            return {
                dialogVisible: false,
                filterText: '',
                dep: {
                    name: '',
                    parentId: -1
                },
                pname: '',
                deps: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },
        watch: {
            filterText(val) {
                this.$refs.tree.filter(val);
            }
        },
        mounted() {
            this.initDeps();
        },
        methods: {
            initDep() {
                this.dep = {
                    name: '',
                    parentId: -1
                }
                this.pname = '';
            },
            addDep2Deps(deps, dep) {
                for (let i = 0; i < deps.length; i++) {
                    let d = deps[i];
                    if (d.id == dep.parentId) {
                        d.children = d.children.concat(dep);
                        if (d.children.length > 0) {
                            d.parent = true;
                        }
                        return;
                    } else {
                        this.addDep2Deps(d.children, dep);
                    }
                }
            },
            doAddDep() {
                this.postRequest("/system/basic/department/", this.dep).then(resp => {
                    if (resp) {
                        this.addDep2Deps(this.deps, resp.obj);
                        this.dialogVisible = false;
                        
                        this.initDep();
                    }
                })
            },
            removeDepFromDeps(p,deps, id) {
                for(let i=0;i<deps.length;i++){
                    let d = deps[i];
                    if (d.id == id) {
                        deps.splice(i, 1);
                        if (deps.length == 0) {
                            p.parent = false;
                        }
                        return;
                    }else{
                        this.removeDepFromDeps(d,d.children, id);
                    }
                }
            },
            deleteDep(data) {
                if (data.parent) {
                    this.$message.error("superior department delete fail");
                } else {
                    this.$confirm('This operation will delete【' + data.name + '】department, do you still continue?', 'Warning', {
                        confirmButtonText: 'confirm',
                        cancelButtonText: 'cancel',
                        type: 'warning'
                    }).then(() => {
                       this.deleteRequest("/system/basic/department/"+data.id).then(resp=>{
                           if (resp) {
                               this.removeDepFromDeps(null,this.deps,data.id);
                           }
                       })
                    }).catch(() => {
                        this.$message({
                            type: 'info',
                            message: 'has cancel delete operation'
                        });
                    });
                }
            },
            showAddDepView(data) {
                this.pname = data.name;
                this.dep.parentId = data.id;
                this.dialogVisible = true;
            },
            initDeps() {
                this.getRequest("/system/basic/department/").then(resp => {
                    if (resp) {
                        this.deps = resp;
                    }
                })
            },
            filterNode(value, data) {
                if (!value) return true;
                return data.name.indexOf(value) !== -1;
            }
        }
    }
</script>

<style>
    .depBtn {
        padding: 2px;
    }
</style>