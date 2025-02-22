<template>
  <div class="mb-6 p-4 bg-white rounded-lg shadow">
    <h2 class="text-xl font-semibold mb-3">导入导出</h2>
    <div class="flex gap-4 mb-4">
      <textarea 
        v-model="importText" 
        class="w-full h-32 border rounded p-2"
        placeholder="请粘贴学生名单，每行一个名字"
      ></textarea>
    </div>
    <div class="flex gap-4 items-center">
      <button 
        @click="handleImport" 
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
      >
        导入名单
      </button>
      <button 
        @click="handleExport" 
        class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
      >
        导出为CSV
      </button>
      <span class="text-gray-600">当前学生数量: {{ studentsCount }}</span>
    </div>
  </div>

  <!-- 模态框 -->
  <Modal
    :show="showModal"
    :type="modalType"
    :title="modalTitle"
    :message="modalMessage"
    :show-confirm="false"
    cancel-text="知道了"
    @close="showModal = false"
  />
</template>

<script setup>
import { ref, computed } from 'vue'
import Modal from './Modal.vue'

const props = defineProps({
  students: {
    type: Array,
    required: true
  },
  db: {
    type: Object,
    required: true
  },
  storeName: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['update-students'])

const importText = ref('')
const studentsCount = computed(() => props.students.length)

// 模态框状态
const showModal = ref(false)
const modalType = ref('info')
const modalTitle = ref('')
const modalMessage = ref('')

// 显示模态框
const showModalMessage = (type, title, message) => {
  modalType.value = type
  modalTitle.value = title
  modalMessage.value = message
  showModal.value = true
}

// 导入学生名单
const handleImport = async () => {
  if (!importText.value.trim()) {
    showModalMessage('warning', '提示', '请先输入学生名单')
    return
  }

  try {
    // 解析输入的文本
    const newNames = importText.value
      .split('\n')
      .map(name => name.trim())
      .filter(name => name)

    if (newNames.length === 0) {
      showModalMessage('warning', '提示', '没有检测到有效的学生名字')
      return
    }

    // 获取当前所有学生
    const currentStudents = await props.db.getAll(props.storeName)
    const currentNames = new Set(currentStudents.map(s => s.name))

    // 找出需要删除的学生
    const namesToDelete = Array.from(currentNames).filter(name => !newNames.includes(name))

    // 删除不在新名单中的学生
    await Promise.all(namesToDelete.map(name => props.db.delete(props.storeName, name)))

    // 准备新学生数据
    const studentsToUpdate = newNames.map(name => ({
      name,
      points: currentNames.has(name) ? 
        currentStudents.find(s => s.name === name).points : 0,
      exchangeCoins: currentNames.has(name) ? 
        currentStudents.find(s => s.name === name).exchangeCoins : 0,
      exchangeAmount: 0
    }))

    // 更新数据库
    await Promise.all(studentsToUpdate.map(student => props.db.put(props.storeName, student)))

    // 更新父组件的学生列表
    emit('update-students', studentsToUpdate)
    importText.value = ''
    showModalMessage('success', '成功', '学生名单已成功导入')
  } catch (error) {
    console.error('导入失败:', error)
    showModalMessage('error', '错误', '导入失败，请检查输入格式')
  }
}

// 导出为CSV
const handleExport = () => {
  if (props.students.length === 0) {
    showModalMessage('warning', '提示', '没有数据可导出')
    return
  }

  try {
    // 准备CSV数据
    const headers = ['姓名', '积分', '兑换币']
    const rows = props.students.map(student => [
      student.name,
      student.points,
      student.exchangeCoins
    ])

    // 生成CSV内容
    const csvContent = [
      headers.join(','),
      ...rows.map(row => row.join(','))
    ].join('\n')

    // 创建Blob对象
    const blob = new Blob(['\ufeff' + csvContent], { 
      type: 'text/csv;charset=utf-8;' 
    })

    // 创建下载链接
    const link = document.createElement('a')
    const url = URL.createObjectURL(blob)
    
    // 设置文件名（使用当前时间戳）
    const now = new Date()
    const timestamp = `${now.getFullYear()}${(now.getMonth()+1).toString().padStart(2,'0')}${now.getDate().toString().padStart(2,'0')}_${now.getHours().toString().padStart(2,'0')}${now.getMinutes().toString().padStart(2,'0')}`
    
    link.setAttribute('href', url)
    link.setAttribute('download', `班级积分表_${timestamp}.csv`)
    link.style.display = 'none'
    
    // 添加到页面并触发下载
    document.body.appendChild(link)
    link.click()
    
    // 清理
    document.body.removeChild(link)
    URL.revokeObjectURL(url)
    showModalMessage('success', '成功', '文件已成功导出')
  } catch (error) {
    console.error('导出失败:', error)
    showModalMessage('error', '错误', '导出失败，请稍后重试')
  }
}
</script>
