<script setup lang="ts">

import { ref, onMounted } from "vue";

interface ContestItem {
  cid: number,
  name: string,
  startTime: number,
  endTime: number,
}

const props = defineProps({
  contestListFunction: Function
});

const tableData = ref<ContestItem[]>([]);

const emit = defineEmits(['response']);

const cb = (e: any) => {
  emit('response', e);
}

onMounted(() => {
  if (props.contestListFunction !== undefined) {
    tableData.value = props.contestListFunction();
  }
});

</script>

<template>
  <div style="height: 99%; width: 90vw; border: 1px solid var(--ep-border-color); border-radius: 3px;">
    <div id="main">

      <el-table align="center" :data="tableData" stripe style="height: 90%;" :border="true">
        <el-table-column sortable align="center" prop="name" label="Contest Name" />
        <el-table-column align="center" label="Progress">
          <template #default="scope">
            <ProgressBar :start-time="tableData[scope.$index].startTime" :end-time="tableData[scope.$index].endTime"/>
          </template>
        </el-table-column>
        <el-table-column align="center" label="Operations" width="100">
          <template #default="scope">
            <el-button size="small" @click="cb(tableData[scope.$index].cid)">View</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
  <div style="width: 100%; height: 60px;"></div>
</template>

<style>
#main {
  width: 100%;
  height: 80%;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
