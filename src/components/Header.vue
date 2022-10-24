<script setup>
import { ref } from "vue";
</script>

<template>
  <n-page-header subtitle="">
    <template #title>
      <a href="/" style="text-decoration: none; color: inherit">
        <n-text depth="3">Are you older than</n-text> <b>18?</b>
      </a>
    </template>
    <template #header> </template>
    <template #avatar>
      <n-avatar :size="48">
        <n-text style="font-size: 20px"> 🔞 </n-text>
      </n-avatar>
    </template>
    <template #extra> </template>
    <template #footer> </template>
  </n-page-header>
</template>

<style scoped></style>
