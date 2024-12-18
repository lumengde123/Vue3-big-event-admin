<script setup>
import { ref, defineEmits, nextTick } from 'vue'
import { Plus } from '@element-plus/icons-vue'
import ChannelSelect from './ChannelSelect.vue'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import { artPublishService, artGetDetailService, artEditService } from '@/api/article'
import { ElMessage } from 'element-plus'
import { baseURL } from '@/utils/request'
import axios from 'axios'

// 默认数据
const defaultForm = {
  title: '',
  cate_id: '',
  cover_img: '',
  content: '',
  state: ''
}

const imgUrl = ref('')
const editorRef = ref()
const visibleDrawer = ref(false)  // 控制抽屉显示隐藏
const formModel = ref({ ...defaultForm })

// 将网络图片地址转换为File对象
async function imageUrlToFile(url, fileName) {
  try {
    // 第一步：使用axios获取网络图片数据
    const response = await axios.get(url, { responseType: 'arraybuffer' });
    const imageData = response.data;

    // 第二步：将图片数据转换为Blob对象
    const blob = new Blob([imageData], { type: response.headers['content-type'] });

    // 第三步：创建一个新的File对象
    const file = new File([blob], fileName, { type: blob.type });

    return file;
  } catch (error) {
    console.error('将图片转换为File对象时发生错误:', error);
    throw error;
  }
}

const open = async (row) => {
  visibleDrawer.value = true
  if (row.id) {
    // console.log('编辑回显')
    const res = await artGetDetailService(row.id)
    formModel.value = res.data.data
    imgUrl.value = baseURL + formModel.value.cover_img
    // 发布是提交的是本地图片文件/ 而编辑中res提供的是图片的链接
    // 提交给后台，需要的是 file 格式的，需要将网络图片下载并转换成 file 格式
    // 网络图片转成 file 对象，需要转换一下
    formModel.value.cover_img = await imageUrlToFile(imgUrl.value, formModel.value.cover_img)
  } else {
    // 添加完成后要对抽屉内容进行重置
    // console.log('添加功能')
    // 这里重置了表单的数据，但是图片地址和富文本编辑器需要手动重置
    formModel.value = { ...defaultForm }
    imgUrl.value = ''
    await nextTick() // 等待 DOM 渲染完再setHTML，不然报错
    editorRef.value.setHTML('')
  }
}

defineExpose({
  open
})

// 上传图片模块
const onUploadFile = (uploadFile) => {
  imgUrl.value = URL.createObjectURL(uploadFile.raw)
  formModel.value.cover_img = uploadFile.raw
}

// 发布文章
const emit = defineEmits(['success'])
const onPublish = async (state) => {
  // 确定是发布还是草稿状态
  formModel.value.state = state

  // 转换 formData 数据
  const fd = new FormData()
  for (let key in formModel.value) {
    fd.append(key, formModel.value[key])
  }

  if (formModel.value.id) {
    // console.log('编辑状态')
    await artEditService(fd)
    ElMessage.success('编辑成功')
    visibleDrawer.value = false
    emit('success', 'edit')
  } else {
    await artPublishService(fd)
    ElMessage.success('添加成功')
    visibleDrawer.value = false
    emit('success', 'add')
  }
}

</script>

<template>
  <!-- 抽屉 -->
  <el-drawer
      v-model="visibleDrawer"
      :title="formModel.id ? '编辑文章' : '添加文章'"
      size="50%"
    >
      <!-- 发表文章表单 -->
      <el-form>
        <el-form-item label="文章标题" prop="title">
          <el-input v-model="formModel.title" placeholder="请输入标题"></el-input>
        </el-form-item>
        <el-form-item label="文章分类" prop="cate_id">
          <channel-select
            v-model="formModel.cate_id"
            width="100%"
          ></channel-select>
        </el-form-item>
        <el-form-item label="文章封面" prop="cover_img">
          <el-upload
            class="avatar-uploader"
            :auto-upload="false"
            :show-file-list="false"
            :on-change="onUploadFile"
          >
            <img v-if="imgUrl" :src="imgUrl" class="avatar" />
            <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
          </el-upload>
        </el-form-item>
        <el-form-item label="文章内容" prop="content">
          <div class="editor">
            <quill-editor
              ref="editorRef"
              theme='snow'
              v-model:content="formModel.content"
              contentType="html"
            >
            </quill-editor>
          </div>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onPublish('已发布')">发布</el-button>
          <el-button type="info" @click="onPublish('草稿')">草稿</el-button>
        </el-form-item>
      </el-form>
    </el-drawer>
</template>

<style lang="scss" scoped>
.avatar-uploader {
  :deep() {
    .avatar {
      width: 178px;
      height: 178px;
      display: block;
    }
    .el-upload {
      border: 1px dashed var(--el-border-color);
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
      transition: var(--el-transition-duration-fast);
    }
    .el-upload:hover {
      border-color: var(--el-color-primary);
    }
    .el-icon.avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 178px;
      height: 178px;
      text-align: center;
    }
  }
}
.editor {
  width: 100%;
  :deep(.ql-editor) {
    min-height: 200px;
  }
}
</style>