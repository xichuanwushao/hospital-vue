<template>
    <el-dialog
        :title="dataForm.workPlanId==null?'新增':'修改'"
        v-if="isAuth(['ROOT', 'SCHEDULE:INSERT', 'SCHEDULE:UPDATE'])"
        :close-on-click-modal="false"
        v-model="visible"
        width="550px"
    >
        <el-form :model="dataForm" ref="dataForm" :rules="dataRule" label-width="80px">
            <el-form-item label="出诊医生" prop="doctorId">
                <el-select v-model="dataForm.doctorId" :disabled="dataForm.workPlanId != null">
                    <el-option v-for="one in doctorList" :label="one.name" :value="one.id"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="出诊时间">
                <div style="width: 100%;">
                    <el-checkbox v-model="checkAll" @change="checkAllChangeHandle">全选</el-checkbox>
                </div>
                <div style="width: 100%;">
                    <el-checkbox-group v-model="checkedSlot">
                        <el-checkbox
                            v-for="(one, index) in slotList"
                            :label="one"
                            :disabled="analyseCheckBoxDisable(index + 1)"
                        />
                    </el-checkbox-group>
                </div>
            </el-form-item>
            <el-form-item label="时段人数">
                <el-slider
                    v-model="dataForm.slotMaximum"
                    :min="1"
                    :max="10"
                    show-input
                    :disabled="dataForm.workPlanId != null"
                />
            </el-form-item>
        </el-form>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="visible = false">取消</el-button>
                <el-button type="primary" @click="dataFormSubmit">确定</el-button>
            </span>
        </template>
    </el-dialog>
</template>

<script>
import dayjs from 'dayjs';
import { ElMessage } from 'element-plus';
export default {
    data: function() {
        return {
            visible: false,
            doctorList: [],
            checkAll: false,
            slotList: [
                '08:00~08:30',
                '08:30~09:00',
                '09:00~09:30',
                '09:30~10:00',
                '10:00~10:30',
                '10:30~11:00',
                '11:00~11:30',
                '11:30~12:00',
                '13:00~13:30',
                '13:30~14:00',
                '14:00~14:30',
                '14:30~15:00',
                '15:00~15:30',
                '16:00~16:30',
                '16:30~17:00'
            ],
            checkedSlot: [],
            oldSlots: [],
            dataForm: {
                workPlanId: null,
                deptSubId: null,
                doctorId: null,
                date: new dayjs().format('YYYY-MM-DD'),
                slots: [],
                slotMaximum: 3
            },
            dataRule: {
                doctorId: [
                    {
                        required: true,
                        message: '出诊医生不能为空'
                    }
                ]
            }
        };
    },
    methods: {
        loadDoctorList: function() {
            let that = this;
            let data = {
                deptSubId: that.dataForm.deptSubId
            };
            that.$http('/doctor/searchByDeptSubId', 'POST', data, true, function(resp) {
                that.doctorList = resp.result;
            });
        },
        reset: function() {
            this.checkAll = false;
            this.checkedSlot = [];
            this.oldSlots = [];
            let dataForm = {
                deptSubId: null,
                doctorId: null,
                date: new dayjs().format('YYYY-MM-DD'),
                slots: [],
                slotMaximum: 3
            };
            this.dataForm = dataForm;
        },
        init: function(workPlanId, deptSubId, date) {
            let that = this;
            that.reset();
            that.dataForm.workPlanId = workPlanId
            that.dataForm.deptSubId = deptSubId;
            that.dataForm.date = date;
            that.visible = true;
            that.$nextTick(() => {
                that.$refs['dataForm'].resetFields();
                that.loadDoctorList();
                if (workPlanId != null) {
                    //TODO 加载出诊计划
                }
            });
        },
        checkAllChangeHandle: function(val) {
            this.checkedSlot = val ? this.slotList : [];
        },
        analyseCheckBoxDisable: function(slot) {
        },
        dataFormSubmit: function() {
            let that = this;
            that.$refs['dataForm'].validate(function(valid) {
                if (valid) {
                    if (that.checkedSlot.length == 0) {
                        ElMessage({
                            message: '出诊时间段没有选择',
                            type: 'warning'
                        });
                        return;
                    }
                    //新增数据
                    if (that.dataForm.workPlanId == null) {
                        //把选中的时段转换成具体编号
                        that.dataForm.slots.length = 0;
                        for (let one of that.checkedSlot) {
                            let index = that.slotList.indexOf(one) + 1;
                            that.dataForm.slots.push(index);
                        }
                        let data = {
                            deptSubId: that.dataForm.deptSubId,
                            doctorId: that.dataForm.doctorId,

                            date: that.dataForm.date,
                            slotMaximum: that.dataForm.slotMaximum,
                            totalMaximum: that.checkedSlot.length * that.dataForm.slotMaximum,
                            slots: that.dataForm.slots
                        };
                        that.$http('/medical/dept/sub/work_plan/insert', 'POST', data, true, function(resp) {
                        let result = resp.result;
                        if (result == '') {
                            ElMessage({
                                message: '操作成功',
                                type: 'success',
                                duration: 1200
                            });
                            that.visible = false;
                            that.$emit('refreshDataList');
                        } else {
                            ElMessage({
                                message: result,
                                type: 'warning',
                                duration: 1200
                            });
                        }
                    });
                }
                //TODO 修改数据
            }
        });
}
    }
};
</script>

<style lang="less" scoped="scoped"></style>
