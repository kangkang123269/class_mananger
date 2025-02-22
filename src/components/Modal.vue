<template>
  <Transition name="modal">
    <div v-if="show" class="fixed inset-0 z-50 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
      <div class="flex items-end sm:items-center justify-center min-h-full p-4 text-center sm:p-0">
        <!-- 背景遮罩 -->
        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity backdrop-blur-sm" 
          aria-hidden="true" 
          @click="$emit('close')"
        ></div>

        <!-- 模态框内容 -->
        <div class="relative inline-block bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:max-w-lg sm:w-full scale-100 opacity-100">
          <!-- 头部 -->
          <div class="px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-start">
              <!-- 图标 -->
              <div class="mx-auto flex-shrink-0 flex items-center justify-center h-12 w-12 rounded-full sm:mx-0 sm:h-10 sm:w-10"
                :class="[typeClasses, 'transform transition-transform hover:scale-110']">
                <component :is="icon" class="h-6 w-6 text-white animate-appear" aria-hidden="true" />
              </div>
              <!-- 内容 -->
              <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left flex-1">
                <h3 class="text-lg leading-6 font-semibold text-gray-900" id="modal-title">
                  {{ title }}
                </h3>
                <div class="mt-2">
                  <p class="text-sm text-gray-500">
                    {{ message }}
                  </p>
                </div>
              </div>
            </div>
          </div>
          <!-- 底部按钮 -->
          <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse gap-2">
            <button v-if="showConfirm" type="button"
              class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 text-base font-medium text-white sm:ml-3 sm:w-auto sm:text-sm transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2"
              :class="[typeClasses, 'hover:opacity-90 focus:ring-' + type + '-500']"
              @click="$emit('confirm')"
            >
              {{ confirmText }}
            </button>
            <button type="button"
              class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:w-auto sm:text-sm transition-all duration-200"
              @click="$emit('close')"
            >
              {{ cancelText }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  show: Boolean,
  type: {
    type: String,
    default: 'info',
    validator: (value) => ['success', 'error', 'info', 'warning'].includes(value)
  },
  title: String,
  message: String,
  showConfirm: {
    type: Boolean,
    default: true
  },
  confirmText: {
    type: String,
    default: '确定'
  },
  cancelText: {
    type: String,
    default: '关闭'
  }
})

defineEmits(['close', 'confirm'])

const typeClasses = computed(() => {
  const classes = {
    success: 'bg-green-500',
    error: 'bg-red-500',
    info: 'bg-blue-500',
    warning: 'bg-yellow-500'
  }
  return classes[props.type]
})

const icon = computed(() => {
  return {
    template: props.type === 'success' 
      ? '<svg xmlns="http://www.w3.org/2000/svg" class="animate-success" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>'
      : props.type === 'error'
      ? '<svg xmlns="http://www.w3.org/2000/svg" class="animate-error" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>'
      : props.type === 'warning'
      ? '<svg xmlns="http://www.w3.org/2000/svg" class="animate-warning" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" /></svg>'
      : '<svg xmlns="http://www.w3.org/2000/svg" class="animate-info" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>'
  }
})
</script>

<style scoped>
.modal-enter-active,
.modal-leave-active {
  transition: all 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.animate-appear {
  animation: appear 0.3s ease-out;
}

.animate-success {
  animation: success 0.5s ease-out;
}

.animate-error {
  animation: error 0.5s ease-out;
}

.animate-warning {
  animation: warning 0.5s ease-out;
}

.animate-info {
  animation: info 0.5s ease-out;
}

@keyframes appear {
  from {
    opacity: 0;
    transform: scale(0.5);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes success {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes error {
  0% {
    transform: rotate(-45deg) scale(0);
  }
  50% {
    transform: rotate(0) scale(1.2);
  }
  100% {
    transform: rotate(0) scale(1);
  }
}

@keyframes warning {
  0% {
    transform: translateY(-20px);
    opacity: 0;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes info {
  0% {
    transform: rotate(180deg) scale(0);
  }
  100% {
    transform: rotate(0) scale(1);
  }
}
</style>
