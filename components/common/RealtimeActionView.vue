<script setup lang="ts">
import { useDouSlackingStore } from '~/stores/dou-slacking'
import MusicIcon from '~icons/uil/music-note'

const douSlackingStore = useDouSlackingStore()

const currentTime = ref(0)

setInterval(() => {
  currentTime.value = Date.now()
}, 1000)

const friendTimeString = computed(() => {
  const time = new Date(currentTime.value)
  const prevTime = new Date(douSlackingStore.stats.update)
  const diff = time.getTime() - prevTime.getTime()
  const diffDays = Math.floor(diff / (24 * 3600 * 1000))
  const diffHours = Math.floor(diff / (3600 * 1000))
  const diffMinutes = Math.floor(diff / (60 * 1000))
  const diffSeconds = Math.floor(diff / 1000)
  if (diffDays > 0) {
    return `${diffDays}天前`
  } else if (diffHours > 0) {
    return `${diffHours}小时前`
  } else if (diffMinutes > 0) {
    return `${diffMinutes}分钟前`
  } else if (diffSeconds > 0) {
    return `${diffSeconds}秒前`
  } else {
    return '刚刚'
  }
})
</script>

<template>
  <div v-if="douSlackingStore.connected" class="realtime-wrapper">
    <div v-if="douSlackingStore.stats.media_playing" class="music-icon">
      <MusicIcon />
    </div>
    <div class="icon" :class="[`icon-${douSlackingStore.stats.iconType}`]">
      <template v-if="douSlackingStore.stats.icon?.endsWith('.png')">
        <img
          :src="`https://dou-slacking.daidr.me/public/icons/${douSlackingStore.stats.icon}`"
          alt=""
        />
      </template>
      <template v-else>
        <div>
          {{ douSlackingStore.stats.icon }}
        </div>
      </template>
    </div>
    <div class="tips">
      <div class="status-wrapper">
        <p v-if="!douSlackingStore.stats.idle" class="text-sm text-primary">
          戴兜正在 <b>{{ douSlackingStore.stats.text }}</b>
        </p>
        <p v-else class="text-xs text-right text-primary">戴兜正在休息</p>
        <p class="text-xs text-right text-primary">
          {{ friendTimeString }}
        </p>
      </div>
      <div v-if="douSlackingStore.stats.media_playing" class="split-line"></div>
      <div v-if="douSlackingStore.stats.media_playing" class="media-wrapper">
        <div class="thumbnail">
          <img
            v-if="douSlackingStore.mediaInfo.thumbnail"
            :src="douSlackingStore.mediaInfo.thumbnail"
          />
          <div v-else>
            <MusicIcon />
          </div>
        </div>
        <div class="media-info">
          <div class="title">
            {{ douSlackingStore.mediaInfo.title }}
          </div>
          <div class="artist">
            {{ douSlackingStore.mediaInfo.artist }}
          </div>
          <div class="time">
            {{ douSlackingStore.stats.media_curr }} /
            {{ douSlackingStore.stats.media_total }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@keyframes shake {
  0% {
    transform: scale(1);
    opacity: 0;
  }
  10% {
    opacity: 0.5;
  }
  50%,
  100% {
    transform: scale(1.3);
    opacity: 0;
  }
}
.realtime-wrapper {
  @apply relative;

  .music-icon {
    @apply absolute z-11 -top-1 -right-1;
    @apply flex items-center justify-center;
    @apply w-13px h-13px text-primary p-1px;
    @apply bg-white rounded-full ring-1 ring-primary-light;
  }

  .icon {
    @apply relative;

    & > img,
    & > div {
      @apply relative z-10 pointer-events-none;
    }

    & > div {
      @apply flex items-center justify-center;
      @apply bg-primary-light;
    }

    &::before {
      @apply absolute content-empty top-0 left-0 right-0 bottom-0 ring-1.5 ring-current z-0;
      animation: shake 2s infinite;
    }

    &.icon-rect {
      &,
      img,
      div {
        @apply w-8 h-8;
        @apply rounded-lg;
      }

      &::before {
        @apply rounded-lg text-primary-light;
      }
    }

    &.icon-circle {
      &,
      img,
      div {
        @apply w-8 h-8;
        @apply rounded-full;
      }

      &::before {
        @apply rounded-full text-primary-light;
      }
    }

    &.icon-point {
      @apply w-4 h-4;
      @apply rounded-full;
      @apply bg-green-6;

      &::before {
        @apply rounded-full text-green-6;
      }
    }
  }

  .tips {
    @apply pointer-events-none select-none;
    @apply absolute mt-1 right-0 whitespace-nowrap p-2;
    @apply bg-white rounded-xl;
    @apply ring-primary-extralight ring-1;
    @apply shadow-lg shadow-primary-extralight;
    @apply opacity-0 transform-gpu translate-y-10 scale-140 blur-xl;
    @apply transition-all duration-500;

    .split-line {
      @apply w-full h-2px my-2 rounded-3px;
      @apply bg-primary-extralight;
    }
  }

  &:hover {
    .tips {
      @apply pointer-events-auto;
      @apply opacity-100 scale-100 blur-0 translate-y-0;
    }
  }

  .media-wrapper {
    @apply flex;

    .thumbnail {
      @apply bg-primary-medium text-white;
      @apply rounded-lg;
      @apply border-3 border-primary-light;
      img {
        @apply rounded-lg;
      }
      div {
        @apply flex items-center justify-center;
        @apply h-full w-full;
      }
      @apply w-3em h-3em mr-1;
    }

    .media-info {
      @apply flex flex-col items-start pointer-events-none;

      & > * {
        @apply text-xs text-right text-primary max-w-10em;
        @apply whitespace-nowrap overflow-hidden text-ellipsis;
      }

      .title {
        @apply font-bold text-sm;
      }

      .time {
        @apply self-end opacity-50;
      }
    }
  }
}
</style>
