<script setup>

// const showLegal = ref(false);
</script>

<template>
  <v-app>
    <v-main>
      <PasswordGenerator />
    </v-main>
    <v-footer app color="background" class="justify-space-between">
      <v-row align="center">
        <v-col class="text-caption">
          &copy; 2024 Julian Christov - made with 💖 in Vienna
        </v-col>

        <v-col cols="auto" class="text-caption" align="end">
          <v-dialog max-width="600">
            <template v-slot:activator="{ props: activatorProps }">
              <v-btn v-bind="activatorProps" size="x-small" text="Legal information" variant="text"
                class="text-caption text-transform-none" prepend-icon="mdi-scale-balance" />
            </template>
            <template v-slot:default="{ isActive }">
              <v-card class="text-body-2 text-justify pa-8">
                <div class="text-h5">Legal information</div>

                <div class="text-h6 mt-4">Terms of Use</div>
                <p>By using our app, you agree to the following terms. This app generates strong random passwords or
                  passphrases. We do not collect, store, or transmit any user data. Use the generated passwords at your
                  own risk.</p>

                <div class="text-h6 mt-4">Privacy Policy</div>
                <p>Your privacy is important to us. We do not collect, store, or share any personal information. All
                  password generation occurs locally on your device.</p>

                <div class="text-h6 mt-4">Data Protection</div>
                <p>We do not handle any user data. All operations are performed locally on your device, ensuring
                  complete privacy and security.</p>

                <div class="text-h6 mt-4">Cookies</div>
                <p>We do not use cookies or any other tracking technologies in this app.</p>

                <div class="text-h6 mt-4">Licensing</div>
                <p>This app is available on GitHub under the MIT License. You are free to use, modify, and distribute
                  the
                  app in accordance with the terms of the MIT License.</p>

                <div align="end" class="mt-4">
                  <v-btn variant="text" text="Close" @click="isActive.value = false" />
                </div>
              </v-card>
            </template>
          </v-dialog>

          <v-btn size="x-small" text="Github source" variant="text" class="text-caption text-transform-none"
            prepend-icon="mdi-github" href="https://github.com/jchristov/pwdify" />
        </v-col>
      </v-row>
    </v-footer>
  </v-app>
</template>

<style>
body {
  font-size: 3.2vw;
}

.text-transform-none {
  text-transform: none;
}
</style>