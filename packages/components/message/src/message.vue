<template>
  <transition
    name="el-message-fade"
    @before-leave="onClose"
    @after-leave="$emit('destroy')"
  >
    <div
      v-show="visible"
      :id="id"
      :class="[
        'el-message',
        type && !icon ? `el-message--${type}` : '',
        center ? 'is-center' : '',
        showClose ? 'is-closable' : '',
        customClass,
      ]"
      :style="customStyle"
      role="alert"
      @mouseenter="clearTimer"
      @mouseleave="startTimer"
    >
      <el-icon v-if="iconComponent" class="el-message__icon" :class="typeClass">
        <component :is="iconComponent" />
      </el-icon>
      <slot>
        <p v-if="!dangerouslyUseHTMLString" class="el-message__content">
          {{ message }}
        </p>
        <!-- Caution here, message could've been compromised, never use user's input as message -->
        <p v-else class="el-message__content" v-html="message"></p>
      </slot>
      <el-icon
        v-if="showClose"
        class="el-message__closeBtn"
        @click.stop="close"
      >
        <close />
      </el-icon>
    </div>
  </transition>
</template>
<script lang="ts">
import { defineComponent, computed, ref, onMounted } from 'vue'
import { useEventListener, useTimeoutFn } from '@vueuse/core'
import { EVENT_CODE } from '@element-plus/utils/aria'
import { ElIcon } from '@element-plus/components/icon'
import { TypeComponents, TypeComponentsMap } from '@element-plus/utils/icon'

import { messageEmits, messageProps } from './message'

import type { CSSProperties } from 'vue'

export default defineComponent({
  name: 'ElMessage',

  components: {
    ElIcon,
    ...TypeComponents,
  },

  props: messageProps,
  emits: messageEmits,

  setup(props) {
    const visible = ref(false)
    let stopTimer: (() => void) | undefined = undefined

    const typeClass = computed(() => {
      const type = props.type
      return type && TypeComponentsMap[type] ? `el-message-icon--${type}` : ''
    })

    const iconComponent = computed(() => {
      return props.icon || TypeComponentsMap[props.type] || ''
    })

    const customStyle = computed<CSSProperties>(() => ({
      top: `${props.offset}px`,
      zIndex: props.zIndex,
    }))

    function startTimer() {
      if (props.duration > 0) {
        ;({ stop: stopTimer } = useTimeoutFn(() => {
          if (visible.value) close()
        }, props.duration))
      }
    }

    function clearTimer() {
      stopTimer?.()
    }

    function close() {
      visible.value = false
    }

    function keydown({ code }: KeyboardEvent) {
      if (code === EVENT_CODE.esc) {
        // press esc to close the message
        if (visible.value) {
          close()
        }
      } else {
        startTimer() // resume timer
      }
    }

    onMounted(() => {
      startTimer()
      visible.value = true
    })

    useEventListener(document, 'keydown', keydown)

    return {
      typeClass,
      iconComponent,
      customStyle,
      visible,

      close,
      clearTimer,
      startTimer,
    }
  },
})
</script>
