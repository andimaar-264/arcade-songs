<script setup lang="ts">
/* eslint-disable import/first, import/no-duplicates */
import { ref, useMeta as useHead, useContext } from '@nuxtjs/composition-api';
import confetti from 'canvas-confetti';
import sleep from 'sleep-promise';
import useGtag from '~/composables/useGtag';
import useDarkMode from '~/composables/useDarkMode';
import useGameInfo from '~/composables/useGameInfo';
import useSheetDialog from '~/composables/useSheetDialog';
import sites from '~/data/sites.json';
import { RICK_SHEET } from '~/utils';

const context = useContext();
const gtag = useGtag();
const { isDarkMode } = useDarkMode();
const { gameCode } = useGameInfo();
const { viewSheet } = useSheetDialog();

const toggleDuration = 5000;
const toggle = ref(isDarkMode.value);
const toggleTimeoutId = ref<number | undefined>(undefined);

async function toggleLightSwitch(buttonPressed: boolean) {
  if (Math.random() < 1) return;

  window.clearTimeout(toggleTimeoutId.value);
  toggleTimeoutId.value = undefined;

  toggle.value = buttonPressed ? !isDarkMode.value : isDarkMode.value;

  toggleTimeoutId.value = window.setTimeout(async () => {
    if (isDarkMode.value === toggle.value) return;

    isDarkMode.value = toggle.value;

    if (isDarkMode.value) {
      await sleep(1000);
      viewSheet(RICK_SHEET);
      confetti({ particleCount: 100, spread: 60, origin: { y: 0.6 }, zIndex: 999 });
      gtag('event', 'SecretFound', { gameCode: gameCode.value, eventSource: 'IndexPage', no: 1 });
    }
  }, toggleDuration);
}

useHead(() => ({
  titleTemplate: '%s',
  title: `${context.$config.siteTitle}`,
}));
</script>

<script lang="ts">
import { defineComponent } from '@nuxtjs/composition-api';

export default defineComponent({
  name: 'IndexPage',
  head: {},
});
</script>

<template>
  <v-container class="fill-height pa-4 pa-sm-8" :class="{ 'dark-mode': isDarkMode }">
    <v-row class="align-center">
      <v-col cols="12" md="6" class="text-center" style="user-select: none;">
        <link v-if="toggle" rel="preload" :href="RICK_SHEET.imageUrl" as="image">
        <v-icon
          :size="120"
          class="MagicLogo__icon my-6"
          :class="{ 'MagicLogo__icon--active': toggle, 'MagicLogo__icon--dark': isDarkMode }"
          :style="{ 'transition': `transform ${toggleDuration}ms !important` }"
          @pointerdown="toggleLightSwitch(true);"
          @pointerup="toggleLightSwitch(false);"
          @pointerleave="toggleLightSwitch(false);"
        >
          mdi-music-box-multiple
        </v-icon>
        <h1 class="mb-5">
          <span>maimai SBY Competition</span>
        </h1>
        <p v-text="$t('page.index.description')" />
      </v-col>
      <v-col cols="12" md="6">
        <v-list
          nav
          max-width="500px"
          class="mx-auto"
        >
          <v-list-item
            v-for="(site, i) in sites.filter((e) => !e.isHidden || isDarkMode)"
            :key="i"
            :to="{ name: 'gameCode', params: { gameCode: site.gameCode }}"
            class="SiteList__item px-4"
          >
            <v-list-item-icon>
              <v-icon :color="site.themeColor">
                mdi-music-box-multiple
              </v-icon>
            </v-list-item-icon>
            <v-list-item-title>
              {{ '12 Oktober 2024' }}
            </v-list-item-title>
            <v-list-item-icon class="SiteList__arrow">
              <v-icon>mdi-arrow-right</v-icon>
            </v-list-item-icon>
          </v-list-item>
        </v-list>
      </v-col>
    </v-row>
  </v-container>
</template>

<style lang="scss">
// .fill-height {
//   background-image: url("/assets/images/maxresdefault.jpg");
//   background-size: cover;
//   background-position: center;
//   background-repeat: no-repeat;
// }
.fill-height {
  background-image: url("@/assets/images/maxresdefault.jpg") !important;
  background-size: cover;
  background-position: center;
  background-color: var(--background-color) !important;
  transition: background-color 0.3s ease;
}

:root {
  --background-color: #fff; // Light mode background color
}

.dark-mode {
  --background-color: #333; // Dark mode background color
}

.SiteList {
  &__arrow {
    opacity: 0;
    transition: opacity 0.5s;
  }

  &__item:hover &__arrow {
    opacity: 1;
  }
}

.MagicLogo {
  &__icon {
    &::before {
      transition: color 1000ms;
    }

    &::after {
      display: none !important;
    }

    &--dark {
      &::before {
        color: yellow;
      }
    }

    &--active {
      transform: rotateZ(+5.125turn);
    }
  }
}
</style>
