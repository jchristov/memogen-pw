<template>
  <v-container fluid class="mb-6 fill-height">
    <v-row>
      <v-col align-self="center">
        <v-card min-height="500" elevation="2" rounded="xl" color="surface-light" class="pa-8">
          <div class="pb-4">
            <h1 class="text-center"><span class="text-primary">Memogen.pw</span> Secure Password Generator</h1>
            <h4 class="text-center">Generate strong, unique and memorable passwords</h4>
          </div>
          <v-btn v-if="0" @click="toggleTheme()" class="float-end">Toggle theme</v-btn>
          <v-row>
            <v-col cols="auto">
              <div>
                <v-btn-toggle v-model="passwordSettings.type" class="ma-2 bg-surface" color="primary" density="compact"
                  mandatory shaped rounded="xl">
                  <v-btn size="small" rounded="xl" variant="elevated">Random</v-btn>
                  <v-btn size="small" rounded="xl" variant="elevated">Memorable</v-btn>
                </v-btn-toggle>
                <div class="d-flex align-center">
                  <v-checkbox v-model="passwordSettings.includeLowercase" label="Include Lowercase" class="ma-2"
                    color="primary" hide-details readonly />
                  <span class="ml-auto font-weight-black text-h5" :class="getColorClass('a')">abc</span>
                </div>
                <div class="d-flex align-center">
                  <v-checkbox v-model="passwordSettings.includeUppercase" label="Include Uppercase" class="ma-2"
                    color="primary" hide-details @change="passwordsGenerated && generatePasswords()" />
                  <span class="ml-auto font-weight-black text-h5" :class="getColorClass('A')">ABC</span>
                </div>
                <div class="d-flex align-center">
                  <v-checkbox v-model="passwordSettings.includeNumbers" label="Include Numbers" class="ma-2"
                    color="primary" hide-details @change="passwordsGenerated && generatePasswords()" />
                  <span class="ml-auto font-weight-black text-h5" :class="getColorClass('1')">123</span>
                </div>
                <div class="d-flex align-center">
                  <v-checkbox v-model="passwordSettings.includeSymbols" label="Include Symbols" class="ma-2"
                    color="primary" hide-details @change="passwordsGenerated && generatePasswords()" />
                  <span class="ml-auto font-weight-black text-h5" :class="getColorClass('.')">./-</span>
                </div>
                <div class="d-flex align-center">
                  <v-select v-model="passwordSettings.separator" class="ma-2" color="primary" label="Separator"
                    :items="separators" variant="solo-filled" hide-details
                    @update:modelValue="passwordsGenerated && generatePasswords()" />
                </div>
              </div>
            </v-col>
            <v-col>
              <v-sheet color="surface" class="pa-8" rounded="xl" height="100%">
                <div v-for=" (password, pIdx) in generatedPasswords" :key="pIdx" class="password">
                  <v-btn icon="mdi-content-copy" variant="text" @click="copyPassword(password)" v-if="isSupported" />
                  <span v-for="(char, cIdx) in password" :key="cIdx" :class="getColorClass(char)" v-html="char" />
                </div>
              </v-sheet>
            </v-col>
          </v-row>
          <div class="mt-4 text-center">
            <v-btn block color="primary" size="large" prepend-icon="mdi-lock-reset" @click="generatePasswords">Generate
              Passwords</v-btn>
          </div>
        </v-card>
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
import { reactive, ref } from 'vue';
import { useTheme } from 'vuetify';
import dictionary from './dictionary';

const { text, isSupported, copy } = useClipboard()
const permissionRead = usePermission('clipboard-read')
const permissionWrite = usePermission('clipboard-write')

const isDark = useDark({ selector: '.v-application' })
const toggleDark = useToggle(isDark)

const theme = useTheme()

function toggleTheme() {
  theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'
}

const separators = [{ title: 'None', value: '' }, { title: 'Space', value: ' ' }, { title: 'Dash', value: '-' }, { title: 'Underscore', value: '_' }, { title: 'Period', value: '.' }, { title: 'Comma', value: ',' }]

const passwordSettings = reactive({
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
// const generatedPasswords = computed(() => generatePasswords())

function getColor(char) {
  if (/[a-z]/.test(char)) return 'yellow';
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

function generatePasswords() {
  const passwordCount = 5;
  const passwords = [];
  for (let i = 0; i < passwordCount; i++) {
    const password = generatePassword()
    passwords.push(password);
  }
  // console.log(passwords);
  generatedPasswords.value = passwords;
  snackbarMessage.value = 'Passwords successfully generated';
  snackbar.value = true;
  passwordsGenerated.value = true;
}

function copyPassword(password) {
  copy(password);
  snackbarMessage.value = 'Password copied to clipboard';
  snackbar.value = true;
}
</script>

<style>
.password {
  font-family: monospace;
  font-size: 1.5rem;
  font-weight: 700;
}
</style>
