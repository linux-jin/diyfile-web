<script setup lang="ts">
import type { UploadFileInfo, UploadInst } from 'naive-ui'
import axios from 'axios'
import { API_URL } from '../../../../../config/config'
import { ResultEnum } from '~/enums/httpEnum'
import { getBackupFile } from '~/api/modules/monitor'

const { t } = useI18n()
const message = useMessage()
const dialog = useDialog()
const user = useUserStore()
const dialogVisible = ref<boolean>(false)
const fileListLengthRef = ref(0)
const uploadRef = ref<UploadInst | null>(null)

const handleBackupFile = () => {
  getBackupFile().then((res) => {
    const jsonObj = JSON.parse(res.data.toString() || '{}')
    // 创建一个包含 json 对象的二进制对象，并指定类型为 application/json
    const blob = new Blob([JSON.stringify(jsonObj)], { type: 'application/json' })
    // 创建一个指向 Blob 对象的临时 URL
    const url = URL.createObjectURL(blob)
    // 创建一个链接到临时 URL 的 a 标签，并设置 download 属性为文件名
    const a = document.createElement('a')
    a.href = url
    const now = new Date()
    a.download = `backup-${now.getFullYear()}-${(now.getMonth() + 1)}-${now.getDay()}.json`
    // 触发 a 标签的点击事件，开始下载文件
    a.click()
    // 释放临时 URL
    URL.revokeObjectURL(url)
  }).catch((err) => {
    console.log(err)
  })
}

const onRequestUpload = (option: any) => {
  const { file } = option
  // 创建 FormData 对象
  const formData = new FormData()
  formData.append('file', file.file)

  // 发送 POST 请求上传文件
  axios.post(`${API_URL}/monitor/restoreData`, formData, {
    headers: {
      'Content-Type': 'multipart/form-data',
      'Authorization': `Bearer ${user.token}`,
    },
  }).then((res) => {
    if (res.data.code === ResultEnum.SUCCESS) {
      message.success(res.data.message)
      return Promise.resolve(res.data)
    }
    if (res.data.code === ResultEnum.UNAUTHORIZED) {
      message.error('登陆已过期，请重新登陆！')
      user.setToken('')
      user.setUserName('')
      user.setAvatar('')
      window.location.href = '/@login'
      return Promise.reject(res.data)
    }
    // 没有权限（code == 403）
    if (res.data.code === ResultEnum.FORBIDDEN) {
      message.error(res.data.message)
      return Promise.reject(res.data)
    }
  }).catch((err) => {
    console.log(err)
  })
}

const onConfirmUpload = () => {
  dialogVisible.value = false
  uploadRef.value?.submit()
}

const handleFileChange = (options: { fileList: UploadFileInfo[] }) => {
  fileListLengthRef.value = options.fileList.length
}

const handleSubmitClick = () => {
  dialog.success({
    title: '确定要上传吗？',
    content: '上传备份文件，系统配置将会覆盖，存储数据将会新增！',
    positiveText: '确定',
    negativeText: '算了',
    closable: false,
    maskClosable: false,
    onPositiveClick: () => {
      onConfirmUpload()
    },
  })
}
</script>

<template>
  <n-card content-style="padding: 0;" class="my-0.5">
    <n-page-header :title="t('menu.setting.backup')" class="mx-0.5" />
  </n-card>
  <n-card content-style="padding: 0;" class="box-card h-full w-full overflow-auto" style="height: calc(100% - 4rem); -ms-overflow-style: none;">
    <div class="flex flex-col mx-2 my-2 ma-2">
      <div class="ma-2 w-40">
        <v-btn prepend-icon="cloud_download" variant="text" @click="handleBackupFile">
          {{ t('button.backup') }}
        </v-btn>
      </div>
      <div class="ma-2 w-40">
        <n-upload
          ref="uploadRef"
          :default-upload="false"
          multiple
          :custom-request="onRequestUpload"
          accept=".json"
          @change="handleFileChange"
        >
          <v-btn prepend-icon="note_add" variant="text">
            {{ t('button.selectFile') }}
          </v-btn>
        </n-upload>
      </div>
      <div class="ma-2 w-40">
        <v-btn prepend-icon="cloud_upload" variant="text" @click="handleSubmitClick" :disabled="!fileListLengthRef">
          {{ t('button.restore') }}
        </v-btn>
      </div>
    </div>
  </n-card>
</template>

<route lang="yaml">
meta:
  layout: admin
</route>
