<template>
  <v-container>
    <v-row align="center" class="mt-8">
      <v-col class="text-center">
        <v-img src="@/assets/logo.png" height="64"></v-img>
        <h1> <span class="text-primary">Memogen.pw</span> Secure Password Generator </h1>
        <h4> Generate strong, unique and memorable passwords </h4>
      </v-col>
      <v-btn v-if="0" variant="text" @click="toggleTheme()" class="position-absolute right-0" :icon="themeIcon" />
      <v-btn v-if="0" variant="text" size="small" class="position-absolute right-0" @click="toggleTheme()"
        :prepend-icon="themeIcon" :text="themeName + ' mode'" />
    </v-row>

    <v-row align="center" justify="space-between">
      <v-col>
        <v-btn-toggle v-model="passwordSettings.type" class="ma-2 bg-surface" color="primary" density="compact"
          mandatory rounded="xl">
          <v-btn size="small" rounded="xl" variant="elevated">Random</v-btn>
          <v-btn size="small" rounded="xl" variant="elevated">Memorable</v-btn>
        </v-btn-toggle>
      </v-col>
      <v-col align="end">
        <v-btn variant="text" @click="toggleTheme()" :prepend-icon="themeIcon" :text="themeName + ' mode'" />
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-card color="surface-light" class="pa-4" min-height="80" rounded="lg">
          <div v-for=" (password, pIdx) in generatedPasswords" :key="pIdx" class="password">
            <v-btn icon="mdi-content-copy" variant="text" @click="copyPassword(password)" v-if="isSupported" />
            <span v-for="(char, cIdx) in password" :key="cIdx" :class="getColorClass(char)" v-html="char" />
          </div>
        </v-card>
        <div class="mt-4 text-center">
          <v-btn block color="primary" size="large" prepend-icon="mdi-autorenew" @click="generatePasswords(1)" text="Generate
            New" />
        </div>
      </v-col>
    </v-row>

    <v-row align="center">
      <v-col cols="12" md="6">
        <span class="text-h5">Words: {{ passwordSettings.words }}</span>
        <v-slider v-if="1" v-model="passwordSettings.words" color="primary" dense hide-details :min="1" :max="10"
          :step="1" :thumb-size="20" thumb-label @update:modelValue="generatePasswords()" />
        <v-number-input v-if="0" v-model="passwordSettings.words" control-variant="split" :min="1" :max="10"
          variant="solo" dense />
      </v-col>
      <v-col cols="12" md="6">
        <span class="text-h5">Include</span>
        <div class="d-flex flex-wrap">
          <div class="d-flex align-center">
            <v-checkbox v-model="passwordSettings.includeLowercase" label="Lowercase" hide-details readonly />
            <span class="font-weight-black text-h6 ml-2" :class="getColorClass('a')">abc</span>
          </div>
          <div class="d-flex align-center">
            <v-checkbox v-model="passwordSettings.includeUppercase" label="Uppercase" hide-details
              @change="passwordsGenerated && generatePasswords()" />
            <div class="font-weight-black text-h6 ml-2" :class="getColorClass('A')">ABC</div>
          </div>
          <div class="d-flex align-center">
            <v-checkbox v-model="passwordSettings.includeNumbers" label="Numbers" hide-details
              @change="passwordsGenerated && generatePasswords()" />
            <span class="font-weight-black text-h6 ml-2" :class="getColorClass('1')">123</span>
          </div>
          <div class="d-flex align-center">
            <v-checkbox v-model="passwordSettings.includeSymbols" label="Symbols" hide-details
              @change="passwordsGenerated && generatePasswords()" />
            <span class="font-weight-black text-h6" :class="getColorClass('.')">./-</span>
          </div>
        </div>
      </v-col>
    </v-row>
    <v-row align="center">
      <v-col cols="12">
        <div class="d-flex align-center">
          <v-select v-model="passwordSettings.separator" class="ma-2" color="primary" label="Separator"
            :items="separators" variant="solo-filled" hide-details
            @update:modelValue="passwordsGenerated && generatePasswords()" />
        </div>
      </v-col>
    </v-row>
  </v-container>

  <v-snackbar v-model="snackbar" color="success" elevation="4" timeout="2000">
    {{ snackbarMessage }}
    <template #actions="{ attrs }">
      <v-btn icon="mdi-close" size="x-small" variant="text" v-bind="attrs" @click="snackbar = false">
        <v-icon>mdi-close</v-icon>
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script setup>
import { useClipboard, useDark, usePermission, useToggle } from '@vueuse/core';
import { computed, reactive, ref } from 'vue';
import { useTheme } from 'vuetify';
import { VNumberInput } from 'vuetify/labs/VNumberInput';
import dictionary from './dictionary';

// Clipboard support
const { text, isSupported, copy } = useClipboard()
const permissionRead = usePermission('clipboard-read')
const permissionWrite = usePermission('clipboard-write')

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

const separators = [{ title: 'None', value: '' }, { title: 'Space', value: ' ' }, { title: 'Dash', value: '-' }, { title: 'Underscore', value: '_' }, { title: 'Period', value: '.' }, { title: 'Comma', value: ',' }]

const passwordSettings = reactive({
  words: 5,
  length: 12,
  type: 1,
  separator: ' ',
  includeLowercase: true,
  includeUppercase: true,
  includeNumbers: false,
  includeSymbols: false,
});

const snackbar = ref(false);
const snackbarMessage = ref('');
const generatedPasswords = ref([]);
const passwordsGenerated = ref(false)

function getColor(char) {
  if (/[a-z]/.test(char)) return 'primary';
  if (/[A-Z]/.test(char)) return 'green';
  if (/[0-9]/.test(char)) return 'blue';
  if (/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(char)) return 'pink';
  return 'grey';
}

function getColorClass(char) {
  return 'text-' + getColor(char);
}

function generatePassword() {
  const wordCount = 5;
  let words = [];
  for (let i = 0; i < wordCount; i++) {
    const wordIndex = Math.floor(Math.random() * dictionary.length);
    let word = dictionary[wordIndex];
    if (passwordSettings.includeUppercase) {
      word = word.charAt(0).toUpperCase() + word.slice(1);
    }
    if (passwordSettings.includeNumbers) {
      word += Math.floor(Math.random() * 10);
    }
    if (passwordSettings.includeSymbols) {
      const symbols = '!@#$%^&*()_+-=[]{};:\'",.<>/?';
      word += symbols[Math.floor(Math.random() * symbols.length)];
    }
    words.push(word);
  }
  return words.join(passwordSettings.separator);
}

function generatePasswords(count = 1) {
  const passwordCount = count;
  const passwords = [];
  console.log('Generating password', count);
  for (let i = 0; i < passwordCount; i++) {
    const password = generatePassword()
    passwords.push(password);
    console.log('Password', password);
  }
  console.log(passwords);
  generatedPasswords.value = passwords;
  // snackbarMessage.value = 'Passwords successfully generated';
  // snackbar.value = true;
  passwordsGenerated.value = true;
}

function copyPassword(password) {
  copy(password);
  snackbarMessage.value = 'Password copied to clipboard';
  snackbar.value = true;
}

generatePasswords(1);
</script>

<style>
.password {
  font-family: monospace;
  font-size: 1.5rem;
  font-weight: 700;
}
</style>
