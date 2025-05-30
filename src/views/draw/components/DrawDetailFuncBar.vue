<script setup lang="ts">
import { NAvatar, NButton, NFlex, NIcon, NImage, NImageGroup, NTag, useMessage } from 'naive-ui'
import { LockClosed24Regular, LockOpen24Regular, Star24Filled, Star24Regular } from '@vicons/fluent'
import { ModelAlt } from '@vicons/carbon'
import { useAuthStore, useDrawStore, useGalleryStore, useUserStore } from '@/store'
import defaultAvatar from '@/assets/avatar.jpg'
import NoPic from '@/assets/no_pic.png'
import api from '@/api'
import { emptyDraw, getRealFileUrl } from '@/utils/functions'

const props = withDefaults(defineProps<Props>(), {
  draw: () => emptyDraw(),
})
const emit = defineEmits<Emit>()
const drawStore = useDrawStore()
const authStore = useAuthStore()
const galleryStore = useGalleryStore()
const userStore = useUserStore()
const ms = useMessage()

interface Emit {
  (e: 'delDraw', uuid: string, prompt: string): void
  (e: 'setPublic', uuid: string, isPublicOrPrivate: boolean): void
}

interface Props {
  draw: Chat.Draw
}
function handleDelDraw(uuid: string, prompt: string) {
  emit('delDraw', uuid, prompt)
}
function handleSetPublic(uuid: string, isPublic: boolean) {
  emit('setPublic', uuid, isPublic)
}
async function handleStar(uuid: string) {
  if (!authStore.token) {
    authStore.setLoginView(true)
    return
  }
  if (props.draw.isStar) {
    const { data } = await api.drawStarOrUnStar<Chat.Draw>(uuid)
    drawStore.updateDraw(data)
    galleryStore.unStarDraw(data)
  } else {
    const { data } = await api.drawStarOrUnStar<Chat.Draw>(uuid)
    drawStore.updateDraw(data)
    galleryStore.starDraw(data)
    ms.success('success')
  }
}
</script>

<template>
  <NFlex vertical>
    <NFlex justify="space-between" align="center">
      <!-- 不可点击按钮组 -->
      <NFlex align="center">
        <NTag size="large" :bordered="false" :color="{ color: '#ff000000' }">
          {{ draw.userName }}
          <template #avatar>
            <NAvatar
              :src="`/api/user/avatar/${draw.userUuid}`" size="large" :fallback-src="defaultAvatar"
              color="#ff0000000"
            />
          </template>
        </NTag>
        <NTag size="medium" :bordered="false" round :color="{ color: '#ff000000' }">
          {{ draw.aiModelName }}
          <template #icon>
            <NIcon :component="ModelAlt" />
          </template>
        </NTag>
      </NFlex>
      <!-- 可点击按钮组 -->
      <NFlex align="center">
        <NTag
          size="medium" :bordered="false" round :color="{ color: '#ff000000' }"
          :checkable="userStore.userInfo.uuid === draw.userUuid" @click="handleSetPublic(draw.uuid, !draw.isPublic)"
        >
          {{ draw.isPublic ? "公开" : "私有" }}
          <template #icon>
            <NIcon :component="draw.isPublic ? LockOpen24Regular : LockClosed24Regular" />
          </template>
        </NTag>
        <NTag
          size="medium" :bordered="false" round :color="{ color: '#ff000000' }" checkable
          @click="handleStar(draw.uuid)"
        >
          {{ draw.starCount }}
          <template #icon>
            <NIcon v-show="!draw.isStar" :component="Star24Regular" />
            <NIcon v-show="draw.isStar" :component="Star24Filled" color="#eac54f" />
          </template>
        </NTag>
      </NFlex>
    </NFlex>
    <NFlex>提示词：{{ draw.prompt }}</NFlex>
    <template v-if="draw.interactingMethod === 4">
      <NFlex>引用图片:</NFlex>
      <NFlex>
        <NImageGroup>
          <NImage
            :src="`${getRealFileUrl(draw.dynamicParams.base_image_url)}?token=${authStore.token}`"
            :fallback-src="NoPic" object-fit="cover" title="原图" width="100"
          />
          <NImage
            v-if="draw.dynamicParams.ref_image_url"
            :src="`${getRealFileUrl(draw.dynamicParams.ref_image_url)}?token=${authStore.token}`" :fallback-src="NoPic"
            object-fit="cover" title="引导图" width="100"
          />
        </NImageGroup>
      </NFlex>
    </template>
    <NFlex justify="end">
      <NButton
        v-show="userStore.userInfo.uuid === draw.userUuid" quaternary type="error"
        @click="handleDelDraw(draw.uuid, draw.prompt)"
      >
        删除
      </NButton>
    </NFlex>
  </NFlex>
</template>
