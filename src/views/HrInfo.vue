<template>
    <div v-if="hr">
        <el-card class="box-card" style="width: 400px">
            <div slot="header" class="clearfix">
                <span>{{hr.name}}</span>
            </div>
            <div>
                <div style="display: flex;justify-content: center">
                    <el-upload
                            :show-file-list="false"
                            :on-success="onSuccess"
                            :data="hr"
                            action="/hr/userface">
                        <img title="click to edit" :src="hr.userface" style="width: 100px;height: 100px;border-radius: 50px"
                             alt="">
                    </el-upload>
                </div>
                <div> <span class="infoTag">Tag：</span>
                    <el-tag type="success" style="margin-right: 5px" v-for="(r,index) in hr.roles" :key="index">
                        {{r.nameZh}}
                    </el-tag>
                </div>
                <div><span class="infoTag">landline number：</span>
                    <el-tag>{{hr.telephone}}</el-tag>
                </div>
                <div><span class="infoTag">phone number：</span>
                    <el-tag>{{hr.phone}}</el-tag>
                </div>
                <div><span class="infoTag">address：</span>
                    <el-tag>{{hr.address}}</el-tag>
                </div>

                <div style="display: flex;justify-content: space-around;margin-top: 10px">
                    <el-button type="primary" @click="showUpdateHrInfoView">edit info</el-button>
                    <el-button type="danger" @click="showUpdatePasswdView">change password</el-button>
                </div>
            </div>
        </el-card>
        <el-dialog
                title="edit user info"
                :visible.sync="dialogVisible"
                width="30%">
            <div>
                <table>
                    <tr>
                        <td class="editTag">
                            <el-tag >Tag:</el-tag>
                        </td>
                        <td>
                            <el-input v-model="hr2.name"></el-input>
                        </td>
                    </tr>
                    <tr>
                        <td class="editTag">
                            <el-tag>landline number：</el-tag>
                        </td>
                        <td>
                            <el-input v-model="hr2.telephone"></el-input>
                        </td>
                    </tr>
                    <tr>
                        <td class="editTag">
                            <el-tag>phone number：</el-tag>
                        </td>
                        <td>
                            <el-input v-model="hr2.phone"></el-input>
                        </td>
                    </tr>
                    <tr>
                        <td class="editTag">
                            <el-tag>address</el-tag>
                        </td>
                        <td>
                            <el-input v-model="hr2.address"></el-input>
                        </td>
                    </tr>
                </table>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">cancel</el-button>
    <el-button type="primary" @click="updateHrInfo">confirm</el-button>
  </span>
        </el-dialog>
        <el-dialog
                title="change password"
                :visible.sync="passwdDialogVisible"
                width="30%">
            <div>
                <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="100px"
                         class="demo-ruleForm">
                    <el-form-item label="input old password" prop="oldpass">
                        <el-input type="password" v-model="ruleForm.oldpass" autocomplete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="input new password" prop="pass">
                        <el-input type="password" v-model="ruleForm.pass" autocomplete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="confirm" prop="checkPass">
                        <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="submitForm('ruleForm')">submit</el-button>
                        <el-button @click="resetForm('ruleForm')">reset</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "HrInfo",
        data() {
            var validatePass = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('please input password'));
                } else {
                    if (this.ruleForm.checkPass !== '') {
                        this.$refs.ruleForm.validateField('checkPass');
                    }
                    callback();
                }
            };
            var validatePass2 = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('please input password again'));
                } else if (value !== this.ruleForm.pass) {
                    callback(new Error('two passwords do not agree!'));
                } else {
                    callback();
                }
            };
            return {
                ruleForm: {
                    oldpass: '',
                    pass: '',
                    checkPass: ''
                },
                rules: {
                    oldpass: [
                        {validator: validatePass, trigger: 'blur'}
                    ],
                    pass: [
                        {validator: validatePass, trigger: 'blur'}
                    ],
                    checkPass: [
                        {validator: validatePass2, trigger: 'blur'}
                    ]
                },
                hr: null,
                hr2: null,
                dialogVisible: false,
                passwdDialogVisible: false
            }
        },
        mounted() {
            this.initHr();
        },
        methods: {
            onSuccess() {
                this.initHr();
            },
            updateHrInfo() {
                this.putRequest("/hr/info", this.hr2).then(resp => {
                    if (resp) {
                        this.dialogVisible = false;
                        this.initHr();
                    }
                })
            },
            showUpdateHrInfoView() {
                this.dialogVisible = true;
            },
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.ruleForm.hrid = this.hr.id;
                        this.putRequest("/hr/pass", this.ruleForm).then(resp => {
                            if (resp) {
                                this.getRequest("/logout");
                                window.sessionStorage.removeItem("user")
                                this.$store.commit('initRoutes', []);
                                this.$router.replace("/");
                            }
                        })
                    } else {
                        return false;
                    }
                });
            },
            resetForm(formName) {
                this.$refs[formName].resetFields();
            },
            showUpdatePasswdView() {
                this.passwdDialogVisible = true;
            },
            initHr() {
                this.getRequest('/hr/info').then(resp => {
                    if (resp) {
                        this.hr = resp;
                        this.hr2 = Object.assign({}, this.hr);
                        window.sessionStorage.setItem("user", JSON.stringify(resp));
                        this.$store.commit('INIT_CURRENTHR', resp);
                    }
                })
            }
        }
    }
</script>

<style scoped>
.infoTag{
    display: inline-block;
        width: 160px;
}
.editTag span{

        display: inline-block;
        width: 120px;
    
}
</style>