<template>
    <div>
        <div style="display: flex;justify-content: space-between">
            <el-button icon="el-icon-plus" type="primary" @click="showAddSalaryView">Add Salary Model</el-button>
            <el-button icon="el-icon-refresh" type="success" @click="initSalaries"></el-button>
        </div>
        <div style="margin-top: 10px">
            <el-table :data="salaries" border stripe>
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column width="120" prop="name" label="Salary model name"></el-table-column>
                <el-table-column width="70" prop="basicSalary" label="Base wage"></el-table-column>
                <el-table-column width="70" prop="trafficSalary" label="Traffic allowance"></el-table-column>
                <el-table-column width="70" prop="lunchSalary" label="Lunch allowance"></el-table-column>
                <el-table-column width="70" prop="bonus" label="Bonus"></el-table-column>
                <el-table-column width="100" prop="createDate" label="Begin Time"></el-table-column>
                <el-table-column label="Pension" align="center">
                    <el-table-column width="70" prop="pensionPer" label="ratio"></el-table-column>
                    <el-table-column width="70" prop="pensionBase" label="base"></el-table-column>
                </el-table-column>
                <el-table-column label="Health insurance" align="center">
                    <el-table-column width="70" prop="medicalPer" label="ratio"></el-table-column>
                    <el-table-column width="70" prop="medicalBase" label="base"></el-table-column>
                </el-table-column>
                <el-table-column label="Housing fund" align="center">
                    <el-table-column width="70" prop="accumulationFundPer" label="ratio"></el-table-column>
                    <el-table-column width="70" prop="accumulationFundBase" label="base"></el-table-column>
                </el-table-column>
                <el-table-column label="Operation">
                    <template slot-scope="scope">
                        <el-button @click="showEditSalaryView(scope.row)">Edit</el-button>
                        <el-button type="danger" @click="deleteSalary(scope.row)">Delete</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <el-dialog
                :title="dialogTitle"
                :visible.sync="dialogVisible"
                width="50%">
            <div style="display: flex;justify-content: space-around;align-items: center">
                <el-steps direction="vertical" :active="activeItemIndex">
                    <el-step :title="itemName" v-for="(itemName,index) in salaryItemName" :key="index"></el-step>
                </el-steps>
                <el-input v-model="salary[title]" :placeholder="'Please input'+salaryItemName[index]+'...'"
                          v-for="(value,title,index) in salary"
                          :key="index" v-show="activeItemIndex==index" style="width: 200px"></el-input>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button @click="preStep">{{activeItemIndex==10?'cancel':'last step'}}</el-button>
    <el-button type="primary" @click="nextStep">{{activeItemIndex==10?'finish':'next step'}}</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "SalSob",
        data() {
            return {
                dialogVisible: false,
                dialogTitle: 'add salary model',
                salaries: [],
                activeItemIndex: 0,
                salaryItemName: [
                    'Base wage',
                    'Traffic allowance',
                    'Lunch allowance',
                    'Bonus',
                    'Pension ratio',
                    'Pension base',
                    'Health insurance ratio',
                    'Health insurance base',
                    'Housing fund ratio',
                    'Housing fund base',
                    'Salary model name'
                ],
                salary: {
                    basicSalary: 0,
                    trafficSalary: 0,
                    lunchSalary: 0,
                    bonus: 0,
                    pensionPer: 0,
                    pensionBase: 0,
                    medicalPer: 0,
                    medicalBase: 0,
                    accumulationFundPer: 0,
                    accumulationFundBase: 0,
                    name: ''
                }
            }
        },
        mounted() {
            this.initSalaries();
        },
        methods: {
            showEditSalaryView(data) {
                this.dialogTitle = 'edit salary model';
                this.dialogVisible = true;
                this.salary.basicSalary = data.basicSalary;
                this.salary.trafficSalary = data.trafficSalary;
                this.salary.lunchSalary = data.lunchSalary;
                this.salary.bonus = data.bonus;
                this.salary.pensionPer = data.pensionPer;
                this.salary.pensionBase = data.pensionBase;
                this.salary.medicalPer = data.medicalPer;
                this.salary.medicalBase = data.medicalBase;
                this.salary.accumulationFundPer = data.accumulationFundPer;
                this.salary.accumulationFundBase = data.accumulationFundBase;
                this.salary.name = data.name;
                this.salary.id = data.id;
            },
            deleteSalary(data) {
                this.$confirm('This operation will delete 【' + data.name + '】，do you still continue？', 'Notification', {
                    cancelButtonText: 'cancel',
                    confirmButtonText: 'confirm'
                }).then(() => {
                    this.deleteRequest("/salary/sob/" + data.id).then(resp => {
                        if (resp) {
                            this.initSalaries();
                        }
                    })
                }).catch(() => {
                    this.$message.info("cancel delete!");
                })
            },
            preStep() {
                if (this.activeItemIndex == 0) {
                    return;
                } else if (this.activeItemIndex == 10) {
                    //close 
                    this.dialogVisible = false;
                    return;
                }
                this.activeItemIndex--;
            },
            nextStep() {
                if (this.activeItemIndex == 10) {
                    if (this.salary.id) {
                        this.putRequest("/salary/sob/", this.salary).then(resp=>{
                            if (resp) {
                                this.initSalaries();
                                this.dialogVisible = false;
                            }
                        })
                    } else {
                        this.postRequest("/salary/sob/", this.salary).then(resp => {
                            if (resp) {
                                this.initSalaries();
                                this.dialogVisible = false;
                            }
                        });
                    }
                    return;
                }
                this.activeItemIndex++;
            },
            showAddSalaryView() {
                //initialize
                this.salary = {
                    basicSalary: 0,
                    trafficSalary: 0,
                    lunchSalary: 0,
                    bonus: 0,
                    pensionPer: 0,
                    pensionBase: 0,
                    medicalPer: 0,
                    medicalBase: 0,
                    accumulationFundPer: 0,
                    accumulationFundBase: 0,
                    name: 'default name'
                }
                this.dialogTitle = 'add salary model';
                this.activeItemIndex = 0;
                this.dialogVisible = true;
            },
            initSalaries() {
                this.getRequest("/salary/sob/").then(resp => {
                    if (resp) {
                        this.salaries = resp;
                    }
                })
            }
        }
    }
</script>

<style scoped>

</style>