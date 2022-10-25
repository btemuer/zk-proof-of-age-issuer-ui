<script setup>
import { ref } from "vue";
</script>

<template>
  <br /><br />
  <div style="max-width: 700px">
    <n-space vertical>
      <n-tag :size="'large'" style="padding: 50px" :bordered="false" round>
        <n-text
          type="tertiary"
          :strong="true"
          depth="3"
          style="font-size: 20px"
        >
        </n-text>
      </n-tag>
      <br />
      <n-h1>
        <n-text type="primary">
          Do you want to prove you are older than 18?
        </n-text>
      </n-h1>
      <n-h4>
        <n-text type="info">
          You may want to do this for any kind of activity on the internet
          requiring age verification, like shopping for liquor.
          <br />
          Yet, you don't want to reveal any information about yourself,
          including your age.
          <br />
          You can do this on the
          <n-text type="primary"><b>Mina blockchain!</b></n-text>
        </n-text>
      </n-h4>
    </n-space>
  </div>
  <br /><br /><n-divider /><br /><br />
</template>

<style scoped></style>
