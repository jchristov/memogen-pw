<template>
  <v-container>
    <v-row align="center" class="mt-4 mt-sm-8">
      <v-col align="center">
        <div class="d-flex ga-4">
          <span><v-img src="@/assets/logo.png" width="64" height="64" /></span>
          <div>
            <div class="text-h5 text-sm-h4"> <span class="text-primary">Memogen.pw</span> Secure Password Generator
            </div>
            <div class="text-subtitle-2 text-sm-subtitle-1"> Generate strong, unique and memorable passwords </div>
          </div>
        </div>
      </v-col>
    </v-row>

    <v-row align="center" justify="space-between">
      <v-col>
        <v-btn-toggle v-model="passwordSettings.typeIndex" color="primary" density="compact" mandatory
          variant="elevated" base-color="surface-light" rounded="xl" @update:modelValue="generate()">
          <v-btn v-for="type in types" :key="type" :text="type" />
        </v-btn-toggle>
      </v-col>
      <v-col align="end">
        <ThemeSwitcher />
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <PasswordView :value="password" />
        <v-btn block color="primary" size="large" prepend-icon="mdi-autorenew" class="mt-4" @click="generate()">Generate
          New {{ type }}</v-btn>
      </v-col>

      <v-col cols="12" md="4" lg="3" v-if="password">
        <SecurityIndicator :value="password" />
      </v-col>
    </v-row>

    <v-row align="start">
      <v-col cols="auto" v-if="passwordSettings.typeIndex === 0">
        <div class="d-flex align-center">
          <div class="text-h6 mr-4">Words</div>
          <v-number-input v-model="passwordSettings.wordsCount" control-variant="split" :min="1"
            :max="passwordSettings.maxWordsCount" density="compact" variant="outlined" max-width="150" hide-details
            @update:modelValue="generate()" />
        </div>
      </v-col>
      <v-col cols="auto" v-else>
        <div class="d-flex align-center">
          <div class="text-h6 mr-4">Characters</div>
          <v-number-input v-model="passwordSettings.passwordLength" control-variant="split" :min="1"
            :max="passwordSettings.maxPasswordLength" density="compact" variant="outlined" max-width="150" hide-details
            @update:modelValue="generate()" />
        </div>
      </v-col>
      <v-col cols="auto" v-if="passwordSettings.typeIndex === 0">
        <div class="d-flex align-center">
          <div class="text-h6 mr-4">Separator</div>
          <v-btn-toggle v-model="passwordSettings.separatorIndex" density="compact" variant="outlined" divided>
            <v-btn v-for="(separator, sIdx) in separators" :key="sIdx" :text="separator.title" />
          </v-btn-toggle>
        </div>
      </v-col>
      <v-col cols="auto">
        <div class="d-flex align-center">
          <div class="text-h6 mr-4">Include</div>
          <v-btn-toggle v-model="passwordSettings.includesIndex" multiple density="compact" variant="outlined" divided>
            <v-btn @click="passwordSettings.includeUppercase = !passwordSettings.includeUppercase">
              <span class="font-weight-black" :class="getColorClass('A')">A-Z</span>
            </v-btn>
            <v-btn @click="passwordSettings.includeNumbers = !passwordSettings.includeNumbers">
              <span class="font-weight-black" :class="getColorClass('1')">0-9</span>
            </v-btn>
            <v-btn @click="passwordSettings.includeSymbols = !passwordSettings.includeSymbols">
              <span class="font-weight-black" :class="getColorClass('.')">./-</span>
            </v-btn>
          </v-btn-toggle>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { computed, reactive, ref } from 'vue';
import { VNumberInput } from 'vuetify/labs/VNumberInput';
import dictionary from './dictionary';
import PasswordView from './PasswordView.vue';
import SecurityIndicator from './SecurityIndicator.vue';
import ThemeSwitcher from './ThemeSwitcher.vue';


const types = ['Passphrase', 'Password']
const type = computed(() => types[passwordSettings.typeIndex])

const separators = [{ title: '˽', value: ' ' }, { title: '-', value: '-' }, { title: '_', value: '_' }, { title: '.', value: '.' }, { title: ',', value: ',' }, { title: '!', value: '!' }, { title: '?', value: '?' }, { title: '#', value: '#' }]
const separator = computed(() => passwordSettings.separatorIndex != null ? separators[passwordSettings.separatorIndex].value : '')

const passwordValue = ref([])
const passphraseWords = ref([]) // Passphrase words in lowercase
const passphraseWordsExtended = computed(() => {
  let words = [];
  for (let word of passphraseWords.value) {
    if (passwordSettings.includesIndex.includes(0)) { // Uppercase
      word = word.charAt(0).toUpperCase() + word.slice(1);
    }
    words.push(word);
  }
  if (passwordSettings.includesIndex.includes(1)) { // Numbers
    const pos = Math.floor(Math.random() * words.length)
    words[pos] += Math.floor(Math.random() * 10);
  }
  if (passwordSettings.includesIndex.includes(2)) { // Symbols
    const pos = Math.floor(Math.random() * words.length)
    const symbols = '!@#$%^&*()_+-=[]{};:\'",.<>/?';
    words[pos] += symbols[Math.floor(Math.random() * symbols.length)];
  }
  return words;
}) // Passphrase words with settings applied
const passphrase = computed(() => passphraseWordsExtended.value.join(separator.value)) // Full passphrase with separators

const password = computed(() => passwordSettings.typeIndex === 0 ? passphrase.value : passwordValue.value.join(''))

const passwordSettings = reactive({
  typeIndex: 0,
  wordsCount: 3,
  maxWordsCount: 10,
  passwordLength: 16,
  maxPasswordLength: 32,
  separatorIndex: 1,
  includesIndex: [0],
  includeLowercase: true,
  includeUppercase: true,
  includeNumbers: false,
  includeSymbols: false,
});

function getColor(char) {
  if (/[A-Z]/.test(char)) return 'green';
  if (/[0-9]/.test(char)) return 'blue';
  if (/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(char)) return 'pink';
  return '';
}

function getColorClass(char) {
  return 'text-' + getColor(char);
}

function generatePassphraseWords() {
  let words = [];
  for (let i = 0; i < passwordSettings.wordsCount; i++) {
    const wordIndex = Math.floor(Math.random() * dictionary.length);
    const word = dictionary[wordIndex];
    words.push(word);
  }
  passphraseWords.value = words
  // console.log('Passphrase:', words);
  return words;
}

function generatePassword() {
  const lowercase = 'abcdefghijklmnopqrstuvwxyz';
  const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const numbers = '0123456789';
  const symbols = '!@#$%^&*()_+-=[]{};:\'",.<>/?';
  const characters = lowercase + (passwordSettings.includeUppercase ? uppercase : '') + (passwordSettings.includeNumbers ? numbers : '') + (passwordSettings.includeSymbols ? symbols : '');
  const password = [];
  for (let i = 0; i < passwordSettings.passwordLength; i++) {
    const charIndex = Math.floor(Math.random() * characters.length);
    password.push(characters[charIndex]);
  }
  passwordValue.value = password;
  // console.log('Password:', password);
  return password;
}

function generate() {
  type.value === 'Passphrase' ? generatePassphraseWords() : generatePassword();
}

generate();
</script>
