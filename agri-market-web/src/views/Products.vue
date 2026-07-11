<template>
  <div>
    <el-card shadow="never">
      <template #header>
        <div class="card-header">
          <span>农产品列表</span>
          <el-button type="primary" :icon="Plus" @click="openCreate">上架农产品</el-button>
        </div>
      </template>

      <el-form :inline="true" class="filter">
        <el-form-item label="分类">
          <el-select v-model="filters.categoryId" placeholder="全部分类" clearable style="width: 160px" @change="loadProducts">
            <el-option v-for="c in categories" :key="c.id" :label="c.name" :value="c.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="filters.status" placeholder="全部状态" clearable style="width: 140px" @change="loadProducts">
            <el-option label="上架" :value="1" />
            <el-option label="下架" :value="0" />
          </el-select>
        </el-form-item>
        <el-form-item label="关键词">
          <el-input v-model="filters.keyword" placeholder="商品名称" clearable @keyup.enter="loadProducts" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :icon="Search" @click="loadProducts">查询</el-button>
          <el-button :icon="Refresh" @click="resetFilter">重置</el-button>
        </el-form-item>
      </el-form>

      <el-table :data="products" v-loading="loading" border stripe>
        <el-table-column prop="id" label="ID" width="60" />
        <el-table-column label="封面" width="84">
          <template #default="{ row }">
            <el-image
              v-if="row.cover"
              :src="row.cover"
              :preview-src-list="[row.cover]"
              fit="cover"
              preview-teleported
              style="width: 48px; height: 48px; border-radius: 4px"
            />
            <span v-else class="no-img">无图</span>
          </template>
        </el-table-column>
        <el-table-column prop="name" label="商品名称" min-width="140" />
        <el-table-column label="分类" width="110">
          <template #default="{ row }">{{ row.category?.name || '-' }}</template>
        </el-table-column>
        <el-table-column label="产地" min-width="140">
          <template #default="{ row }">{{ row.origin?.name || '-' }}</template>
        </el-table-column>
        <el-table-column label="价格" width="100">
          <template #default="{ row }">¥{{ row.price }}/{{ row.unit || '' }}</template>
        </el-table-column>
        <el-table-column prop="stock" label="库存" width="80" />
        <el-table-column prop="sales" label="销量" width="80" />
        <el-table-column label="状态" width="90">
          <template #default="{ row }">
            <el-tag :type="row.status === 1 ? 'success' : 'info'">{{ row.status === 1 ? '上架' : '下架' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="240" fixed="right">
          <template #default="{ row }">
            <el-button size="small" :icon="Edit" @click="openEdit(row)">编辑</el-button>
            <el-button size="small" :type="row.status === 1 ? 'warning' : 'success'" @click="toggleStatus(row)">
              {{ row.status === 1 ? '下架' : '上架' }}
            </el-button>
            <el-button size="small" type="danger" :icon="Delete" @click="remove(row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog v-model="dialogVisible" :title="editingId ? '编辑农产品' : '上架农产品'" width="560px">
      <el-form :model="form" label-width="90px" :rules="rules" ref="formRef">
        <el-form-item label="名称" prop="name">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="分类">
          <el-select v-model="form.categoryId" placeholder="请选择分类" style="width: 100%">
            <el-option v-for="c in categories" :key="c.id" :label="c.name" :value="c.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="产地">
          <el-select v-model="form.originId" placeholder="请选择产地" style="width: 100%">
            <el-option v-for="o in origins" :key="o.id" :label="o.name" :value="o.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="价格" prop="price">
          <el-input-number v-model="form.price" :min="0" :precision="2" :step="1" />
        </el-form-item>
        <el-form-item label="库存" prop="stock">
          <el-input-number v-model="form.stock" :min="0" :step="1" />
        </el-form-item>
        <el-form-item label="单位">
          <el-input v-model="form.unit" placeholder="如：斤/公斤/份" style="width: 160px" />
        </el-form-item>
        <el-form-item label="封面图">
          <div class="cover-uploader">
            <el-image v-if="form.cover" :src="form.cover" fit="cover" class="cover-preview" />
            <div class="cover-actions">
              <el-upload
                action="/api/upload"
                accept="image/*"
                :show-file-list="false"
                :before-upload="beforeCoverUpload"
                :on-success="handleCoverSuccess"
              >
                <el-button :icon="Upload">{{ form.cover ? '更换图片' : '上传图片' }}</el-button>
              </el-upload>
              <el-button v-if="form.cover" text type="danger" @click="form.cover = ''">移除图片</el-button>
            </div>
          </div>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="form.description" type="textarea" :rows="2" />
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="form.status">
            <el-radio :value="1">上架</el-radio>
            <el-radio :value="0">下架</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submit">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus, Search, Refresh, Edit, Delete, Upload } from '@element-plus/icons-vue'
import { productApi, categoryApi, originApi } from '../api'

const products = ref([])
const categories = ref([])
const origins = ref([])
const loading = ref(false)
const dialogVisible = ref(false)
const editingId = ref(null)
const formRef = ref()

const filters = reactive({ categoryId: null, status: null, keyword: '' })
const form = reactive({ name: '', categoryId: null, originId: null, price: 0, stock: 0, unit: '', cover: '', description: '', status: 1 })
const rules = {
  name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
  price: [{ required: true, message: '请输入价格', trigger: 'blur' }]
}

const loadProducts = async () => {
  loading.value = true
  try {
    products.value = await productApi.list(filters)
  } finally {
    loading.value = false
  }
}
const loadOptions = async () => {
  categories.value = await categoryApi.list()
  origins.value = await originApi.list()
}
const resetFilter = () => {
  Object.assign(filters, { categoryId: null, status: null, keyword: '' })
  loadProducts()
}
const resetForm = () => {
  Object.assign(form, { name: '', categoryId: null, originId: null, price: 0, stock: 0, unit: '', cover: '', description: '', status: 1 })
}
const openCreate = () => {
  editingId.value = null
  resetForm()
  dialogVisible.value = true
}
const openEdit = (row) => {
  editingId.value = row.id
  Object.assign(form, {
    name: row.name,
    categoryId: row.category?.id || null,
    originId: row.origin?.id || null,
    price: row.price,
    stock: row.stock,
    unit: row.unit,
    cover: row.cover,
    description: row.description,
    status: row.status
  })
  dialogVisible.value = true
}
const submit = async () => {
  await formRef.value.validate()
  if (editingId.value) {
    await productApi.update(editingId.value, form)
    ElMessage.success('已更新')
  } else {
    await productApi.create(form)
    ElMessage.success('上架成功')
  }
  dialogVisible.value = false
  loadProducts()
}
const toggleStatus = async (row) => {
  const target = row.status === 1 ? 0 : 1
  await productApi.changeStatus(row.id, target)
  ElMessage.success(target === 1 ? '已上架' : '已下架')
  loadProducts()
}
const remove = async (row) => {
  await ElMessageBox.confirm(`确定删除「${row.name}」吗？`, '提示', { type: 'warning' })
  await productApi.remove(row.id)
  ElMessage.success('已删除')
  loadProducts()
}

// 图片上传：校验类型与大小
const beforeCoverUpload = (file) => {
  const isImage = file.type && file.type.startsWith('image/')
  const isLt5M = file.size / 1024 / 1024 < 5
  if (!isImage) ElMessage.error('只能上传图片文件')
  if (!isLt5M) ElMessage.error('图片大小不能超过 5MB')
  return isImage && isLt5M
}
// 上传成功：后端返回 { code, data: { url } }（el-upload 不走 axios 拦截器，需自行解析）
const handleCoverSuccess = (res) => {
  if (res && res.code === 0) {
    form.cover = res.data.url
    ElMessage.success('图片上传成功')
  } else {
    ElMessage.error((res && res.message) || '图片上传失败')
  }
}

onMounted(() => {
  loadOptions()
  loadProducts()
})
</script>

<style scoped>
.card-header { display: flex; justify-content: space-between; align-items: center; }
.filter { margin-bottom: 8px; }
.no-img { color: #bbb; font-size: 12px; }
.cover-uploader { display: flex; flex-direction: column; align-items: flex-start; gap: 8px; }
.cover-preview { width: 120px; height: 120px; border-radius: 6px; border: 1px solid #ebeef5; }
.cover-actions { display: flex; align-items: center; gap: 8px; }
</style>
