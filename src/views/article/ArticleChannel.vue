<script setup>
import {
  Edit,
  Delete
} from '@element-plus/icons-vue'
import { ref } from 'vue'
import { artDelChannelService, artGetChannelsService } from '@/api/article'
import ChannelEdit from './components/ChannelEdit.vue'
import { ElMessage, ElMessageBox } from 'element-plus'

const channelList = ref([])
const loading = ref(false) // 表格加载的loading动画
const dialog = ref()

// 获取文章分类数据
const getChannelList = async () => {
  loading.value = true
  const res = await artGetChannelsService()
  channelList.value = res.data.data
  loading.value = false
}
getChannelList()

// 添加分类按钮
const onAddChannel = () => {
  dialog.value.open({})
}

// 编辑按钮
const onEditChannel = (row) => {
  dialog.value.open(row)
}

// 删除按钮
const onDelChannel = async (row) => {
  await ElMessageBox.confirm('你确认删除该分类信息吗？', '温馨提示', {
    type: 'warning',
    confirmButtonText: '确认',
    cancelButtonText: '取消'
  })
  await artDelChannelService(row.id)
  ElMessage({ type: 'success', message: '删除成功', plain: true })
  getChannelList()
}

// 监听success事件，调用回显
const onSuccess = () => {
  getChannelList()
}

</script>

<template>
  <page-container title="文章分类">
    <template #extra>
      <el-button type="primary" @click="onAddChannel">添加分类</el-button>
    </template>

    <el-table v-loading="loading" :data="channelList" style="width: 100%">
      <el-table-column label="序号" width="100" type="index"> </el-table-column>
      <el-table-column label="分类名称" prop="cate_name"></el-table-column>
      <el-table-column label="分类别名" prop="cate_alias"></el-table-column>
      <el-table-column label="操作" width="100">
        <template #default="{ row }">
          <el-button
            :icon="Edit"
            circle
            plain
            type="primary"
            @click="onEditChannel(row)"
          ></el-button>
          <el-button
            :icon="Delete"
            circle
            plain
            type="danger"
            @click="onDelChannel(row)"
          ></el-button>
        </template>
      </el-table-column>
      <template #empty>
        <el-empty description="没有数据" />
      </template>
    </el-table>

    <channel-edit ref="dialog" @success="onSuccess"></channel-edit>
    
  </page-container>

</template>