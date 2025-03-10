<template>
  <div id="App-Wrapper" :class="[appWrapperClass, theme]" :style="wrapperStyle">
    <div
      id="App-Container"
      class="app-container max-w-10/12 lg:max-w-screen-2xl px-3 lg:px-8"
      @keydown.meta.k.stop.prevent="handleOpenModal"
      tabindex="-1"
      :style="cssVariables"
    >
      <HeaderMain />
      <div class="app-banner app-banner-image" :style="headerImage" />
      <div class="app-banner app-banner-screen" :style="headerBaseBackground" />
      <div class="relative z-10">
        <router-view v-slot="{ Component }">
          <transition name="fade-slide-y" mode="out-in">
            <component :is="Component" />
          </transition>
        </router-view>
      </div>
    </div>
    <div id="loading-bar-wrapper" :class="loadingBarClass"></div>
  </div>
  <Footer :style="cssVariables" />
  <div class="App-Mobile-sidebar" v-if="isMobile">
    <div id="App-Mobile-Profile" class="App-Mobile-wrapper">
      <MobileMenu />
    </div>
  </div>
  <Navigator />
  <Dia v-if="!isMobile && configReady" />
  <teleport to="head">
    <title>{{ title }}</title>
  </teleport>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  onBeforeMount,
  onMounted,
  onUnmounted,
  ref,
  watch
} from 'vue'
import { useAppStore } from '@/stores/app'
import { useCommonStore } from '@/stores/common'
import { useMetaStore } from '@/stores/meta'
import { useSearchStore } from './stores/search'
import HeaderMain from '@/components/Header/src/Header.vue'
import Footer from '@/components/Footer.vue'
import Navigator from '@/components/Navigator.vue'
import MobileMenu from '@/components/MobileMenu.vue'
import Dia from '@/components/Dia.vue'
import defaultCover from '@/assets/default-cover.jpg'
import { useI18n } from 'vue-i18n'

export default defineComponent({
  name: 'App',
  components: {
    HeaderMain,
    Footer,
    Navigator,
    MobileMenu,
    Dia
  },
  setup() {
    const appStore = useAppStore()
    const commonStore = useCommonStore()
    const metaStore = useMetaStore()
    const searchStore = useSearchStore()
    const MOBILE_WITH = 996 // Using the mobile width by Bootstrap design.
    const { t } = useI18n()

    const appWrapperClass = 'app-wrapper'
    const loadingBarClass = ref({
      'nprogress-custom-parent': false
    })

    let pagelink = `\n\nRead more at: ${document.location.href}`

    /** Intiallizing App config and other setups */
    const initialApp = async () => {
      initResizeEvent()
      await appStore.fetchConfig().then(() => {
        metaStore.addScripts(appStore.themeConfig.site_meta.cdn.prismjs)
        // Change the favicon dynamically.
        if (
          appStore.themeConfig.site_meta.favicon &&
          appStore.themeConfig.site_meta.favicon !== ''
        ) {
          const link = document.querySelector("link[rel~='icon']")
          if (link)
            link.setAttribute('href', appStore.themeConfig.site_meta.favicon)
        }

        if (appStore.themeConfig.plugins.copy_protection.enable) {
          const locale = appStore.locale
          const linkPlaceholder =
            locale === 'cn'
              ? appStore.themeConfig.plugins.copy_protection.link.cn
              : appStore.themeConfig.plugins.copy_protection.link.en
          const authorPlaceholder =
            locale === 'cn'
              ? appStore.themeConfig.plugins.copy_protection.author.cn
              : appStore.themeConfig.plugins.copy_protection.author.en
          const licensePlaceholder =
            locale === 'cn'
              ? appStore.themeConfig.plugins.copy_protection.license.cn
              : appStore.themeConfig.plugins.copy_protection.license.en

          pagelink = `\n\n---------------------------------\n${authorPlaceholder}: ${appStore.themeConfig.site.author}\n${linkPlaceholder}: ${document.location.href}\n${licensePlaceholder}`
          initialCopyrightScript()
        }
      })
    }

    const copyEventHandler = (event: any) => {
      if (document.getSelection() instanceof Selection) {
        if (document.getSelection()?.toString() !== '' && event.clipboardData) {
          event.clipboardData.setData(
            'text',
            document.getSelection() + pagelink
          )
          event.preventDefault()
        }
      }
    }

    /** Adding copy listner */
    const initialCopyrightScript = () => {
      document.addEventListener('copy', copyEventHandler)
    }

    const isMobile = computed(() => {
      return commonStore.isMobile
    })

    const resizeHandler = () => {
      const rect = document.body.getBoundingClientRect()
      const mobileState = rect.width - 1 < MOBILE_WITH
      if (isMobile.value !== mobileState)
        commonStore.changeMobileState(mobileState)
    }

    const initResizeEvent = () => {
      resizeHandler()
      window.addEventListener('resize', resizeHandler)
    }

    const handleOpenModal = () => {
      searchStore.setOpenModal(true)
    }

    onBeforeMount(initialApp)

    onUnmounted(() => {
      document.removeEventListener('copy', copyEventHandler)
      window.removeEventListener('resize', resizeHandler)
    })

    const wrapperStyle = ref({ 'min-height': '100vh' })

    onMounted(() => {
      let wrapperHeight = screen.height
      const footerEl = document.getElementById('footer')
      const footerHeight = footerEl?.getBoundingClientRect().height
      if (typeof footerHeight === 'number') {
        wrapperHeight = wrapperHeight - footerHeight * 2
      }
      wrapperStyle.value = {
        'min-height': wrapperHeight + 'px'
      }
    })

    /**
     * Watches the app loading status, adding the `nprogress-custom-parent`
     * class to the nprogress container when loading.
     */
    watch(
      () => appStore.appLoading,
      newState => {
        loadingBarClass.value['nprogress-custom-parent'] = newState
      }
    )

    return {
      title: computed(() => metaStore.getTitle),
      theme: computed(() => appStore.theme),
      scripts: computed(() => metaStore.scripts),
      themeConfig: computed(() => appStore.themeConfig),
      headerImage: computed(() => {
        return {
          backgroundImage: `url(${
            commonStore.headerImage
          }), url(${defaultCover})`,
          opacity: commonStore.headerImage !== '' ? 1 : 0
        }
      }),
      headerBaseBackground: computed(() => {
        return {
          background: appStore.themeConfig.theme.header_gradient_css,
          opacity: commonStore.headerImage !== '' ? 0.91 : 0.99
        }
      }),
      wrapperStyle: computed(() => wrapperStyle.value),
      handleEscKey: appStore.handleEscKey,
      isMobile: computed(() => commonStore.isMobile),
      configReady: computed(() => appStore.configReady),
      cssVariables: computed(() => {
        if (appStore.theme === 'theme-dark') {
          return `
            --text-accent: ${appStore.themeConfig.theme.gradient.color_1};
            --text-sub-accent: ${appStore.themeConfig.theme.gradient.color_3};
            --main-gradient: ${appStore.themeConfig.theme.header_gradient_css};
          `
        }
        return `
          --text-accent: ${appStore.themeConfig.theme.gradient.color_3};
          --text-sub-accent: ${appStore.themeConfig.theme.gradient.color_2};
          --main-gradient: ${appStore.themeConfig.theme.header_gradient_css};
        `
      }),
      appWrapperClass,
      loadingBarClass,
      handleOpenModal,
      t
    }
  }
})
</script>

<style lang="scss">
body {
  background: var(--background-primary-alt);
}

*:focus {
  outline: none;
}

#app {
  @apply relative min-w-full min-h-screen h-full;
  font-family: Rubik, Avenir, Helvetica, Arial, sans-serif;
  .app-wrapper {
    @apply bg-ob-deep-900 min-w-full h-full pb-12;
    transition-property: transform, border-radius;
    transition-duration: 350ms;
    transition-timing-function: ease;
    transform-origin: 0 42%;
    .app-container {
      color: var(--text-normal);
      margin: 0 auto;
    }
  }

  .header-wave {
    position: absolute;
    top: 100px;
    left: 0;
    z-index: 1;
  }

  .App-Mobile-sidebar {
    @apply fixed top-0 bottom-0 left-0;
  }
  .App-Mobile-wrapper {
    @apply relative overflow-y-auto h-full -mr-4 pr-6 pl-4 pt-8 opacity-0;
    transition: all 0.85s cubic-bezier(0, 1.8, 1, 1.2);
    transform: translateY(-20%);
    width: 280px;
  }
}

.app-banner {
  content: '';
  display: block;
  height: 600px;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 1;
  clip-path: polygon(
    100% 0,
    0 0,
    0 77.5%,
    1% 77.4%,
    2% 77.1%,
    3% 76.6%,
    4% 75.9%,
    5% 75.05%,
    6% 74.05%,
    7% 72.95%,
    8% 71.75%,
    9% 70.55%,
    10% 69.3%,
    11% 68.05%,
    12% 66.9%,
    13% 65.8%,
    14% 64.8%,
    15% 64%,
    16% 63.35%,
    17% 62.85%,
    18% 62.6%,
    19% 62.5%,
    20% 62.65%,
    21% 63%,
    22% 63.5%,
    23% 64.2%,
    24% 65.1%,
    25% 66.1%,
    26% 67.2%,
    27% 68.4%,
    28% 69.65%,
    29% 70.9%,
    30% 72.15%,
    31% 73.3%,
    32% 74.35%,
    33% 75.3%,
    34% 76.1%,
    35% 76.75%,
    36% 77.2%,
    37% 77.45%,
    38% 77.5%,
    39% 77.3%,
    40% 76.95%,
    41% 76.4%,
    42% 75.65%,
    43% 74.75%,
    44% 73.75%,
    45% 72.6%,
    46% 71.4%,
    47% 70.15%,
    48% 68.9%,
    49% 67.7%,
    50% 66.55%,
    51% 65.5%,
    52% 64.55%,
    53% 63.75%,
    54% 63.15%,
    55% 62.75%,
    56% 62.55%,
    57% 62.5%,
    58% 62.7%,
    59% 63.1%,
    60% 63.7%,
    61% 64.45%,
    62% 65.4%,
    63% 66.45%,
    64% 67.6%,
    65% 68.8%,
    66% 70.05%,
    67% 71.3%,
    68% 72.5%,
    69% 73.6%,
    70% 74.65%,
    71% 75.55%,
    72% 76.35%,
    73% 76.9%,
    74% 77.3%,
    75% 77.5%,
    76% 77.45%,
    77% 77.25%,
    78% 76.8%,
    79% 76.2%,
    80% 75.4%,
    81% 74.45%,
    82% 73.4%,
    83% 72.25%,
    84% 71.05%,
    85% 69.8%,
    86% 68.55%,
    87% 67.35%,
    88% 66.2%,
    89% 65.2%,
    90% 64.3%,
    91% 63.55%,
    92% 63%,
    93% 62.65%,
    94% 62.5%,
    95% 62.55%,
    96% 62.8%,
    97% 63.3%,
    98% 63.9%,
    99% 64.75%,
    100% 65.7%
  );
}

.app-banner-image {
  z-index: 1;
  background-size: cover;
  opacity: 0;
  transition: ease-in-out opacity 300ms;
}

.app-banner-screen {
  transition: ease-in-out opacity 300ms;
  z-index: 2;
  opacity: 0.91;
}
</style>
