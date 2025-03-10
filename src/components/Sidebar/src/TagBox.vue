<template>
  <div class="sidebar-box">
    <SubTitle :title="'titles.tag_list'" icon="tag" />
    <TagList>
      <template v-if="tags && tags.length > 0">
        <TagItem
          v-for="tag in tags"
          :key="tag.slug"
          :name="tag.name"
          :slug="tag.slug"
          :count="tag.count"
          size="xs"
        />
        <div
          class="
            flex flex-row
            items-center
            hover:opacity-50
            mr-2
            mb-2
            cursor-pointer
            transition-all
          "
        >
          <span class="text-center px-3 py-1 rounded-md text-sm">
            <b class="border-b-2 border-ob hover:text-ob">
              <router-link to="/tags">
                {{ t('settings.more-tags') }} ...
              </router-link>
            </b>
          </span>
        </div>
      </template>
      <template v-else-if="tags">
        <ob-skeleton tag="li" :count="10" height="20px" width="3rem" />
      </template>
      <template v-else>
        <div class="flex flex-row justify-center items-center">
          <SvgIcon class="stroke-ob-bright mr-2" icon-class="warning" />
          {{ t('settings.empty-tag') }}
        </div>
      </template>
    </TagList>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted } from 'vue'
import { SubTitle } from '@/components/Title'
import { useTagStore } from '@/stores/tag'
import { TagList, TagItem } from '@/components/Tag'
import { useI18n } from 'vue-i18n'
import SvgIcon from '@/components/SvgIcon/index.vue'

export default defineComponent({
  name: 'ObTag',
  components: { SubTitle, TagList, TagItem, SvgIcon },
  setup() {
    const tagStore = useTagStore()
    const { t } = useI18n()

    const fetchData = async () => {
      tagStore.fetchTagsByCount(10)
    }

    onMounted(fetchData)

    return {
      tags: computed(() => {
        if (tagStore.isLoaded && tagStore.tags.length === 0) return null
        return tagStore.tags
      }),
      t
    }
  }
})
</script>

<style lang="scss">
.sidebar-box li.ob-skeleton {
  @apply mr-2 mb-2;
}
</style>
