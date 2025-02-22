<template>
  <div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-6 text-center">班级积分系统</h1>

    <!-- 批量操作 -->
    <div class="mb-6 p-4 bg-white rounded-lg shadow">
      <h2 class="text-xl font-semibold mb-3">批量操作</h2>
      <div class="flex gap-4">
        <input type="number" v-model="batchPoints" class="border p-2 rounded" placeholder="输入分数">
        <button @click="batchAddPoints" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
          全班加分
        </button>
        <button @click="batchSubtractPoints" class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600">
          全班减分
        </button>
      </div>
    </div>

    <ImportExport :students="students" :db="db" :store-name="storeName" @update-students="updateStudentsList" />

    <!-- 积分排行 -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div v-for="student in sortedStudents" :key="student.name" class="bg-white p-4 rounded-lg shadow">
        <div class="flex justify-between items-center mb-2">
          <h3 class="font-semibold">{{ student.name }}</h3>
          <div class="space-x-2">
            <button @click="addPoints(student)" class="bg-green-500 text-white px-2 py-1 rounded text-sm">
              +1
            </button>
            <button @click="subtractPoints(student)" class="bg-red-500 text-white px-2 py-1 rounded text-sm">
              -1
            </button>
          </div>
        </div>
        <div class="flex justify-between text-sm">
          <span>积分: {{ student.points }}</span>
          <span>兑换币: {{ student.exchangeCoins }}</span>
        </div>
        <div class="mt-2">
          <input type="number" v-model="student.exchangeAmount" class="border p-1 rounded w-20 text-sm"
            placeholder="兑换数量">
          <button @click="exchangeReward(student)" class="bg-blue-500 text-white px-2 py-1 rounded text-sm ml-2">
            兑换奖励
          </button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { openDB } from 'idb'
import ImportExport from './components/ImportExport.vue'

const dbName = 'classPointsDB'
const storeName = 'students'
const db = ref(null)
const students = ref([])
const batchPoints = ref(0)

const studentNames = []

// 初始化数据库
const initDB = async () => {
  db.value = await openDB(dbName, 1, {
    upgrade(db) {
      if (!db.objectStoreNames.contains(storeName)) {
        const store = db.createObjectStore(storeName, { keyPath: 'name' })
        store.createIndex('points', 'points')
      }
    }
  })
}

const updateStudentsList = (newStudents) => {
  students.value = newStudents
}

// 加载学生数据
const loadStudents = async () => {
  const allStudents = await db.value.getAll(storeName)
  if (allStudents.length === 0) {
    // 初始化学生数据
    const initialStudents = studentNames.map(name => ({
      name,
      points: 0,
      exchangeCoins: 0,
      exchangeAmount: 0
    }))
    await Promise.all(initialStudents.map(student =>
      db.value.put(storeName, student)
    ))
    students.value = initialStudents
  } else {
    students.value = allStudents
  }
}

// 更新学生数据
const updateStudent = async (student) => {
  await db.value.put(storeName, { ...student })
}

// 按积分排序的学生列表
const sortedStudents = computed(() => {
  return [...students.value].sort((a, b) => b.points - a.points)
})

// 添加积分
const addPoints = async (student) => {
  const updatedStudent = {
    ...student,
    points: student.points + 1,
    exchangeCoins: student.exchangeCoins + 1
  }
  await updateStudent(updatedStudent)
  const index = students.value.findIndex(s => s.name === student.name)
  if (index !== -1) {
    students.value[index] = updatedStudent
  }
}

// 减少积分
const subtractPoints = async (student) => {
  if (student.points > 0) {
    const updatedStudent = {
      ...student,
      points: student.points - 1,
      exchangeCoins: Math.max(0, student.exchangeCoins - 1)
    }
    await updateStudent(updatedStudent)
    const index = students.value.findIndex(s => s.name === student.name)
    if (index !== -1) {
      students.value[index] = updatedStudent
    }
  }
}

// 批量添加积分
const batchAddPoints = async () => {
  const points = parseInt(batchPoints.value)
  if (points > 0) {
    const updatedStudents = await Promise.all(students.value.map(async student => {
      const updatedStudent = {
        ...student,
        points: student.points + points,
        exchangeCoins: student.exchangeCoins + points
      }
      await updateStudent(updatedStudent)
      return updatedStudent
    }))
    students.value = updatedStudents
    batchPoints.value = 0
  }
}

// 批量减少积分
const batchSubtractPoints = async () => {
  const points = parseInt(batchPoints.value)
  if (points > 0) {
    const updatedStudents = await Promise.all(students.value.map(async student => {
      const updatedStudent = {
        ...student,
        points: Math.max(0, student.points - points),
        exchangeCoins: Math.max(0, student.exchangeCoins - points)
      }
      await updateStudent(updatedStudent)
      return updatedStudent
    }))
    students.value = updatedStudents
    batchPoints.value = 0
  }
}

// 兑换奖励
const exchangeReward = async (student) => {
  const amount = parseInt(student.exchangeAmount)
  if (amount > 0 && amount <= student.exchangeCoins) {
    const updatedStudent = {
      ...student,
      exchangeCoins: student.exchangeCoins - amount,
      exchangeAmount: 0
    }
    await updateStudent(updatedStudent)
    const index = students.value.findIndex(s => s.name === student.name)
    if (index !== -1) {
      students.value[index] = updatedStudent
    }
  }
}

onMounted(async () => {
  await initDB()
  await loadStudents()
})
</script>

<style>
body {
  background-color: #f3f4f6;
}
</style>
