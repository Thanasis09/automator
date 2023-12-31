<script setup lang="ts">
import { onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { RouterView } from 'vue-router'
import { useToast } from 'vue-toast-notification'
import Navigation from '@components/navigation.vue'
import DatabaseController from '@database/controller'
import useCharactersStore from '@stores/characters'
import useModStore from '@stores/mod'
import useThemeStore from '@stores/theme'
import useTraitsStore from '@stores/traits'

const { t } = useI18n()
const $toast = useToast()
const database = DatabaseController.getInstance()
const characterStore = useCharactersStore()
const modStore = useModStore()
const themeStore = useThemeStore()
const traitsStore = useTraitsStore()

onMounted(async () => {
  await database.init()
  characterStore.refresh()

  themeStore.fetch()
  const directory = localStorage.getItem('directory')
  if (directory) {
    modStore.$patch({ directory })
    try {
      await modStore.readEntries(directory)
      const traitsDir = modStore.getCommonDirectory
      if (traitsDir) {
        await traitsStore.readDir(traitsDir.path)
        await traitsStore.fetchFromLocalStorage()
      }
    } catch {
      $toast.error(t('error.generic-dir-read'))
    }
  }
})
</script>

<template>
  <navigation />
  <div class="content px-8">
    <router-view />
  </div>
</template>
