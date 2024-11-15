<script setup>
import { useClipboard, usePermission } from '@vueuse/core';
import { ref } from 'vue';

// Clipboard support
const { copy, isSupported } = useClipboard()
const permissionRead = usePermission('clipboard-read')
const permissionWrite = usePermission('clipboard-write')

const props = defineProps(['value'])

const snackbar = ref(false);
const snackbarMessage = ref('');

function getColor(char) {
  if (/[A-Z]/.test(char)) return 'green';
  if (/[0-9]/.test(char)) return 'blue';
  if (/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(char)) return 'pink';
  return '';
}

function getColorClass(char) {
  return 'text-' + getColor(char);
}

function copyPassword(password) {
  copy(password);
  snackbarMessage.value = 'Password copied to clipboard';
  snackbar.value = true;
}
</script>

<template>
  <v-card class="pa-4" variant="tonal">
    <div class="text-h5 font-weight-black" style="font-family: monospace">
      <v-btn icon="mdi-content-copy" variant="text" @click="copyPassword(props.value)" v-if="isSupported" />
      <span v-for="(char, cIdx) in props.value" :key="cIdx" :class="getColorClass(char)" v-html="char" />
    </div>
    <v-snackbar v-model="snackbar" color="success" elevation="4" location="center" location-strategy="connected"
      target="parent" timeout="1000">
      {{ snackbarMessage }}
      <template #actions="{ attrs }">
        <v-btn icon="mdi-close" size="x-small" variant="text" v-bind="attrs" @click="snackbar = false">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </template>
    </v-snackbar>
  </v-card>
</template>