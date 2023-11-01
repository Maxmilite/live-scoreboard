<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { i18nAssets } from '~/assets/constants';

const props = defineProps({
    contestInfoFunction: Function,
    recordFunction: Function, 
    cid: Number
});

interface ContestItem {
  cid: number,
  name: string,
  startTime: number,
  endTime: number,
}

interface RecordItem {
  uid: number,
  cid: string,
  pid: number,
  recordTime: number,
  score: number,
}

const contestInfo = ref<ContestItem>();
const records = ref<ContestItem>();
const title = ref('');

onMounted(() => {
    contestInfo.value = props.contestInfoFunction!(props.cid);
    title.value = i18nAssets.title + " - " + contestInfo.value?.name;
    records.value = props.recordFunction!(props.cid);
    console.log(records.value)
})

</script>

<template>
  <div style="padding-top: 20px">
    <h1>{{ title }}</h1>
  </div>
</template>

<style>
.ep-button {
  margin: 4px;
}

.ep-button+.ep-button {
  margin-left: 0;
  margin: 4px;
}
</style>
