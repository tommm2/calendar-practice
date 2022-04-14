<script setup>
import { toRefs } from "vue";

const props = defineProps({
  data: Object,
});

const { data } = toRefs(props);

const emit = defineEmits(["handleClose"]);
</script>

<template>
  <div
    class="fixed top-0 w-full flex justify-center items-center h-full z-10 backdrop-blur-sm"
    v-if="data"
    @click.self="emit('handleClose', false)"
  >
    <ul
      class="w-[350px] p-5 bg-white border-t-8 border-gray-800 shadow-md rounded-lg relative space-y-4"
    >
      <li><span>標案名稱</span>: {{ data.headline }}</li>
      <li>標案日期: {{ data.publishdate }}</li>
      <li>標案單位: {{ data.unit }}</li>
      <li>標案代碼: {{ data.job_number }}</li>
      <li>標案狀態: {{ data.status }}</li>
      <li>
        標案詳細連結:
        <a class="underline text-blue-600" target="_blank" :href="data.url">{{
          data.url
        }}</a>
      </li>
      <button
        @click="emit('handleClose', false)"
        class="absolute font-extrabold top-0 right-2"
      >
        X
      </button>
    </ul>
  </div>
</template>
<style scoped></style>
