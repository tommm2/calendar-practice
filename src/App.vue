<script setup>
import { reactive, ref, inject, onMounted } from "vue";
import Popup from "./components/Popup.vue";

import "@fullcalendar/core/vdom";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import Datepicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const axios = inject("axios");

// popup 顯示
const popupVisable = ref(false);

// popup 顯示資料
const PopupData = ref([]);

// 關鍵字選取
const keywordInput = ref("");

// 日期選取
const dateInput = ref(["2021/8/1", "2022/2/28"]);

// 行事曆設置
const calendarOptions = reactive({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  initialDate: "2021-08-01",
  events: [],
  eventClick(info) {
    info.jsEvent.preventDefault();
    getPopupData(info.event.id);
    showPopup(true);
  },
});

// 將日期格式化
const format = (date) => {
  const arr = date.map((item) => {
    if (item) {
      let year = item.getFullYear();
      let month = item.getMonth() + 1;
      let date = item.getDate();
      return `${year}/${month}/${date}`;
    }
    return "";
  });

  if (arr[1])
    getData({ keyword: keywordInput.value, date_range: arr.join("-") });

  return arr.join("-");
};

// 初始化資料
const getData = (options = {}) => {
  const baseUrl = "https://eunomics.net/api/v1/tenders?";
  let params = "";

  Object.keys(options).forEach((key) => {
    if (!options[key]) {
      params += "";
    }
    params += `${key}=${options[key]}&`;
  });

  console.log(baseUrl + params);
  
  axios.get(baseUrl + params).then(({ data }) => {
    let modifiedData = data.map(
      ({ headline: title, publishdate: start, ...rest }) => ({
        title,
        start,
        backgroundColor: "#313131",
        borderColor: "#313131",
        ...rest,
      })
    );

    calendarOptions.events = modifiedData;
  });
};

// 取得單一資料
const getPopupData = (id) => {
  const baseUrl = "https://eunomics.net/api/v1/tenders";
  axios.get(`${baseUrl}/${id}`).then(({ data }) => {
    PopupData.value = data;
  });
};

// 切換 popup modal 狀態
const showPopup = (status) => {
  popupVisable.value = status;
  if (!popupVisable.value) PopupData.value = null;
};

onMounted(() => {
  getData();
});
</script>

<template>
  <div class="p-10">
    <div class="space-y-4 mb-10 flex-col flex justify-center items-center">
      <form
        class="w-full md:w-1/2 lg:w-1/3 flex justify-center items-center relative"
        @submit.prevent="
          getData({ keyword: keywordInput, date_range: dateInput.join('-') })
        "
      >
        <input
          type="text"
          placeholder="請輸入關鍵字..."
          class="rounded-lg px-4 w-full py-2 shadow-md focus:outline-blue-500 border-1 border-[#ddd]"
          v-model="keywordInput"
        />
        <button
          class="text-white bg-[#313131] p-2 rounded-tr-lg rounded-br-lg absolute right-0 shadow-md"
          type="submit"
        >
          搜尋關鍵字
        </button>
      </form>
      <Datepicker
        :format="format"
        placeholder="請選擇日期..."
        class="w-full md:w-1/2 lg:w-1/3 shadow-md"
        v-model="dateInput"
        range
        required
      />
    </div>

    <FullCalendar :options="calendarOptions" />
  </div>

  <transition name="zoom">
    <Popup :data="PopupData" @handle-close="showPopup" v-if="popupVisable" />
  </transition>
</template>

<style>
.zoom-enter-active,
.zoom-leave-active {
  @apply duration-200 ease-in-out;
}

.zoom-enter-from {
  transform: scale(0);
}
.zoom-enter-to {
  transform: scale(1);
}

.zoom-leave-from {
  transform: scale(1);
}
.zoom-leave-to {
  transform: scale(0);
}
</style>
