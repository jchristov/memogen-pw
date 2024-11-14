<script setup>
import { zxcvbn } from '@zxcvbn-ts/core';
import { computed } from 'vue';

const props = defineProps(['value'])

const colors = ['red-darken-4', 'red', 'orange', 'yellow', 'green', 'gray']
const levels = ['Very Weak', 'Weak', 'Fair', 'Good', 'Strong', 'Unknown']

const strengthCheck = computed(() => zxcvbn(props.value)) // Calculate strength
const scoreColor = computed(() => colors[strengthCheck.value.score]) // Get color based on score
const securityLevel = computed(() => levels[strengthCheck.value.score]) // Get security level based on score
</script>

<template>
  <v-card :color="scoreColor" class="pa-6" height="100%" hover>
    <v-row justify="space-between">
      <v-col class="flex-column text-center">
        <div class="text-caption text-uppercase">Security Score</div>
        <div class="text-h4 font-weight-black">{{ strengthCheck.score }} / 4</div>
        <div class="text-h6 font-weight-bold">{{ securityLevel }}</div>
      </v-col>
      <v-col class="flex-column text-caption">
        <div><b>Guesses:</b> {{ strengthCheck.guesses }}</div>
        <div><b>Crack time:</b> {{ strengthCheck.crackTimesDisplay.offlineSlowHashing1e4PerSecond }}</div>
        <div v-if="strengthCheck.feedback.warning"><b>Warning:</b> {{ strengthCheck.feedback.warning }}</div>
        <div v-if="strengthCheck.feedback.suggestions.length"><b>Suggestions:</b>
          <div v-for="suggestion in strengthCheck.feedback.suggestions" class="ml-4">{{ suggestion }}</div>
        </div>
      </v-col>
    </v-row>
    <div v-if="0">
      <div class="d-flex flex-column text-center">
        <span class="text-caption text-uppercase">Security Score</span>
        <span class="text-h4 font-weight-black">{{ strengthCheck.score }} / 4</span>
        <span class="text-h6 font-weight-bold">{{ securityLevel }}</span>
      </div>
      <div class="d-flex flex-column text-caption">
        <div><b>Guesses:</b> {{ strengthCheck.guesses }}</div>
        <div><b>Crack time:</b> {{ strengthCheck.crackTimesDisplay.offlineSlowHashing1e4PerSecond }}</div>
        <div v-if="strengthCheck.feedback.warning"><b>Warning:</b> {{ strengthCheck.feedback.warning }}</div>
        <div v-if="strengthCheck.feedback.suggestions.length"><b>Suggestions:</b>
          <ul class="ml-8">
            <li v-for="suggestion in strengthCheck.feedback.suggestions">{{ suggestion }}</li>
          </ul>
        </div>
      </div>
    </div>
  </v-card>
</template>