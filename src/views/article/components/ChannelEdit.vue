<script setup>
import { artAddChannelService, artEditChannelService } from '@/api/article';
import { ElMessage } from 'element-plus';
import { ref } from 'vue'
const dialogVisible = ref(false)
const formRef = ref(null)

// 重置表单状态
const resetForm = () => {
  if (formRef.value) {
    formRef.value.resetFields(); 
  }
}

const open = async (row) => {
  dialogVisible.value = true
  formModel.value = { ...row }
}

defineExpose({
  open
})

const formModel = ref({
  cate_name: '',
  cate_alias: ''
})

const rules = {
  cate_name: [
    { required: true, message: '请输入分类名称', trigger: 'blur'},
    { 
      pattern: /^\S{1,10}$/, 
      message: '分类名称必须是1~10位的非空字符', 
      trigger: 'blur'}
  ],
  cate_alias: [
    { required: true, message: '请输入分类别名', trigger: 'blur'},
    { 
      pattern: /^[a-zA-Z0-9]{1,15}$/,
      message: '分类别名必须是1~15位字母或数字',
      trigger: 'blur'
    }
  ]
}

// 提交表单信息，通知父组件回显
const emit = defineEmits(['success'])
const onSubmit = async () => {
  await formRef.value.validate()
  formModel.value.id
    ? await artEditChannelService(formModel.value)
    : await artAddChannelService(formModel.value)
  ElMessage({
    type: 'success',
    message: formModel.value.id ? '编辑成功' : '添加成功',
    plain: true
  })
  dialogVisible.value = false
  emit('success')
}


</script>

<template>
  <el-dialog 
    v-model="dialogVisible" 
    :title="formModel.id ? '编辑分类' : '添加分类'" 
    width="30%"
    @closed="resetForm"
  >
    <el-form
      ref="formRef"
      label-width="100px"
      style="padding-right: 30px"
      :model="formModel"
      :rules="rules"
    >
      <el-form-item label="分类名称" prop="cate_name">
        <el-input
          v-model="formModel.cate_name"
          minlength="1"
          maxlength="10"
          placeholder="请输入分类名称"
        ></el-input>
      </el-form-item>
      <el-form-item label="分类别名" prop="cate_alias">
        <el-input
          v-model="formModel.cate_alias"
          minlength="1"
          maxlength="15"
          placeholder="请输入分类别名"
        ></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="onSubmit"> 确认 </el-button>
      </span>
    </template>
  </el-dialog>
</template>