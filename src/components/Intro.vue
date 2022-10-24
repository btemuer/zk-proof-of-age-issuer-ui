<script setup>
import { ref } from "vue";
</script>

<template>
  <br /><br />
  <div style="max-width: 600px">
    <n-space vertical>
      <n-tag :size="'large'" style="padding: 50px" :bordered="false" round>
        <n-text
          type="tertiary"
          :strong="true"
          depth="3"
          style="font-size: 40px"
        >
          x = ?
        </n-text>
      </n-tag>
      <br />
      <n-h1>
        <n-text type="primary"> yz? </n-text>
      </n-h1>
      <n-h4>
        <n-text type="info">
          a
          <n-text type="primary"><b>b</b></n-text
          >. c
        </n-text>
      </n-h4>
    </n-space>
  </div>
  <br /><br /><n-divider /><br /><br />
</template>

<style scoped></style>
