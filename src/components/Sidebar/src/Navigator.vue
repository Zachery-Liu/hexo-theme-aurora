<template>
  <ul
    id="sidebar-navigator"
    class="flex flex-row bg-ob-deep-800 rounded-xl shadow-2xl justify-items-center overflow-hidden"
  >
    <li
      class="border-r-4 border-ob-deep-900 flex justify-center py-3 w-full hover:opacity-50 hover:text-ob transition-all cursor-pointer"
      @click="goBack"
    >
      <SvgIcon class="inline-block text-3xl" icon-class="go-back" />
    </li>
    <li
      class="border-r-4 border-ob-deep-900 flex justify-center py-3 w-full hover:opacity-50 hover:text-ob transition-all cursor-pointer"
      @click="backToTop"
    >
      <SvgIcon class="inline-block text-3xl" icon-class="back-to-top" />
    </li>
    <li
      class="flex justify-center py-3 w-full hover:opacity-50 hover:text-ob transition-all cursor-pointer"
      @click="jumpToComments"
      data-dia="jump-to-comment"
    >
      <SvgIcon class="inline-block text-3xl" icon-class="quote" />
    </li>
  </ul>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import { useRouter } from 'vue-router'
import SvgIcon from '@/components/SvgIcon/index.vue'

export default defineComponent({
  name: 'Navigator',
  components: { SvgIcon },
  setup() {
    const router = useRouter()
    const commentOffset = ref(0)
    const commentEl = ref()

    const backToTop = () => {
      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      })
    }

    const goBack = () => {
      router.back()
    }

    const jumpToComments = () => {
      commentEl.value = document.getElementById('comments')
      if (commentEl.value) {
        // 120 is the height of the header element
        commentOffset.value =
          commentEl.value && commentEl.value instanceof HTMLElement
            ? commentEl.value.offsetTop + 120 - 30
            : 0
      }
      window.scrollTo({
        top: commentOffset.value,
        behavior: 'smooth'
      })
    }

    return {
      goBack,
      backToTop,
      jumpToComments
    }
  }
})
</script>

<style lang="scss" scoped>
#sidebar-navigator {
  svg {
    pointer-events: none;
  }
}
</style>
