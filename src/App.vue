<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import { darkTheme } from "naive-ui";
import header_ from "./components/Header.vue";
import intro_ from "./components/Intro.vue";
import content from "./components/Content.vue";
import footer_ from "./components/Footer.vue";
</script>

<template>
  <n-config-provider :theme="darkTheme">
    <n-loading-bar-provider>
      <n-message-provider placement="top-right">
        <n-notification-provider placement="bottom-right">
          <n-layout>
            <n-layout-header bordered style="padding: 2rem">
              <header_ />
            </n-layout-header>
            <n-layout-content style="padding: 2rem">
              <intro_ />
              <content />
            </n-layout-content>
            <n-layout-footer bordered style="padding: 2rem">
              <footer_ />
            </n-layout-footer>
          </n-layout>
        </n-notification-provider>
      </n-message-provider>
    </n-loading-bar-provider>
  </n-config-provider>
</template>

<style scoped></style>
