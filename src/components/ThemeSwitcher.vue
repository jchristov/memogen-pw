<script setup>
import { useDark, useToggle } from '@vueuse/core';
import { computed } from 'vue';
import { useTheme } from 'vuetify';

// Dark mode support
const isDark = useDark()
const toggleDark = useToggle(isDark)
const theme = useTheme()
const themeName = computed(() => isDark.value ? 'light' : 'dark')
const themeIcon = computed(() => isDark.value ? 'mdi-brightness-7' : 'mdi-brightness-4')

function toggleTheme() {
  toggleDark()
  theme.global.name.value = isDark.value ? 'dark' : 'light'
}
</script>

<template>
  <v-btn variant="text" @click="toggleTheme()" :prepend-icon="themeIcon" :text="themeName + ' mode'" />
</template>