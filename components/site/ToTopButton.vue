<script setup lang="ts">
import IconArrowUp from '~icons/uil/arrow-up'

const props = defineProps<{
  showProcess: boolean
  scrollEl: string
  boundingEl?: string
}>()

const toTop = () => {
  document.querySelector(props.scrollEl)?.scrollTo({
    top: 0,
    behavior: 'smooth',
  })
}

const pathTotalLength = ref(148)
const pathCurrentLength = ref(0)

const setPathTotalLength = (length: number) => {
  pathTotalLength.value = length
}

const { y: scrollY } = useScroll(
  document.querySelector(props.scrollEl) as HTMLElement,
)

if (props.showProcess && props.boundingEl) {
  const { y, height } = useElementBounding(
    document.querySelector(props.boundingEl) as HTMLElement,
  )

  const scrollThrottle = useThrottleFn(newVal => {
    const totalHeight = height.value
    const clientHeight =
      document.querySelector(props.scrollEl)?.clientHeight || 0

    pathCurrentLength.value =
      (newVal / (totalHeight - clientHeight) + 1) * pathTotalLength.value
    if (pathCurrentLength.value > pathTotalLength.value - 1) {
      pathCurrentLength.value = pathTotalLength.value - 1
    } else if (pathCurrentLength.value < 0) {
      pathCurrentLength.value = 0
    }
  }, 10)

  watch(y, scrollThrottle, { immediate: true })
}
</script>

<template>
  <Transition name="fade">
    <div v-if="scrollY > 5" class="to-top-wrapper">
      <div class="to-top-button" @click="toTop">
        <svg v-if="showProcess" class="process-bar" view-box="0 0 48 48">
          <UiStrokedRoundedRect
            :class="{ hide: pathCurrentLength >= pathTotalLength - 1 }"
            :style="{
              strokeDashoffset: pathCurrentLength,
              strokeDasharray: `${pathTotalLength}px ${pathTotalLength}px`,
            }"
            :set-path-total-length="setPathTotalLength"
            :width="48"
            :height="48"
            :stroke-width="5"
            :radius="16"
          />
        </svg>
        <IconArrowUp />
      </div>
    </div>
  </Transition>
</template>

<style lang="scss" scoped>
.to-top-wrapper {
  @apply fixed bottom-8 right-8 w-50px h-50px;
  @apply lg:(bottom-20 right-20);

  .to-top-button {
    @apply relative rounded-2xl bg-white w-12 h-12;
    @apply flex items-center justify-center text-2xl text-primary;
    @apply shadow-md shadow-primary/5 cursor-pointer;
    @apply transition-all;
    @apply rounded-2xl;
    filter: drop-shadow(0 0 1px rgb(var(--color-primary-light)));

    &:hover {
      @apply shadow-xl shadow-primary/15;
      @apply bg-primary;
      @apply text-primary-extralight;

      & .process-bar path {
        @apply stroke-primary-extralight;
      }
    }

    .process-bar {
      @apply absolute -top-0 left-0;
      @apply w-full h-full;

      path {
        @apply stroke-primary fill-none;
        // path端点圆角
        @apply stroke-cap-round transition-colors;

        &.hide {
          @apply opacity-0;
        }
      }
    }
  }
}
</style>
