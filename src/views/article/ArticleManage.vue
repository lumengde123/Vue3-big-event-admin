<script setup>
import { Delete, Edit} from '@element-plus/icons-vue'
import { ref } from 'vue'
import ChannelSelect from './components/ChannelSelect.vue'
import { artGetListService } from '@/api/article'
import { formatTime } from '@/utils/format'

const articleList = ref([]) // 文章列表
const total = ref(0)  // 总条数
const loading = ref(false)

// 定义请求参数对象
const params = ref({
  pagenum: 1,
  pagesize: 5,
  cate_id: '',
  state: ''
})

// 获取文章列表
const getArticleList = async () => {
  loading.value = true
  const res = await artGetListService(params.value)
  articleList.value = res.data.data
  total.value = res.data.total
  loading.value = false
}
getArticleList()

// 条件搜索按钮
const onSearch = () => {
  params.value.pagenum = 1
  getArticleList()
}
// 重置按钮
const onReset = () => {
  params.value.pagenum = 1
  params.value.cate_id = ''
  params.value.state = ''
  getArticleList()
}

// 处理分页逻辑
const onSizeChange = (size) => {
  // console.log('当前每条页数', size)
  // 只要是每页条数变化了，那么原本正在访问的页 意义不大了，数据大概率不在那一页了
  // 所以重新从第一页开始渲染即可
  params.value.pagenum = 1
  params.value.pagesize = size
  // 基于最新的当前页 和 每页条数，渲染数据
  getArticleList()
}
const onCurrentChange = page => {
  // 更新当前页
  params.value.pagenum = page
  // 基于最新的当前页，渲染数据
  getArticleList()
}

// 编辑逻辑
const onEditArticle = (row) => {
  console.log(row)
}

// 删除逻辑
const onDeleteArticle = (row) => {
  console.log(row)
}


</script>

<template>
  <page-container title="文章管理">
    <template #extra>
      <el-button type="primary">发布文章</el-button>
    </template>

    <!-- 主体/表单区域 -->
    <el-form inline>
      <el-form-item label="文章分类：">
        <channel-select v-model="params.cate_id"></channel-select>
      </el-form-item>
      <el-form-item label="发布状态：">
        <el-select v-model="params.state" style="width: 240px">
          <el-option label="已发布" value="已发布"></el-option>
          <el-option label="草稿" value="草稿"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="onSearch" type="primary">搜索</el-button>
        <el-button @click="onReset">重置</el-button>
      </el-form-item>
    </el-form>
    <!-- 主体/表格区域 -->
    <el-table :data="articleList" v-loading="loading">
      <el-table-column label="文章标题" prop="title">
        <template #default="{ row }">
          <el-link type="primary" :underline="false">{{ row.title }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="分类" prop="cate_name"></el-table-column>
      <el-table-column label="发表时间" prop="pub_date">
        <template #default="{ row }">
          {{ formatTime(row.pub_time) }}
        </template>
      </el-table-column>
      <el-table-column label="状态" prop="state"></el-table-column>
      <!-- 利用作用于插槽row 可以获取当前行的数据 => 类似于 v-for 的 item -->
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button 
            circle 
            plain 
            type="primary" 
            :icon="Edit" 
            @click="onEditArticle(row)"
          ></el-button>
          <el-button 
            circle 
            plain 
            type="danger" 
            :icon="Delete"
            @click="onDeleteArticle(row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页区域 -->
    <el-pagination
      v-model:current-page="params.pagenum"
      v-model:page-size="params.pagesize"
      :page-sizes="[3, 5, 10]"
      :background="true"
      layout="jumper, total, sizes, prev, pager, next"
      :total="total"
      @size-change="onSizeChange"
      @current-change="onCurrentChange"
      style="margin-top: 20px; justify-content: flex-end;"
    />
  </page-container>

</template>