<script setup>
import { ref, computed, watch, onMounted } from 'vue';

import SearchCom from './components/SearchCom.vue';
import GroupCom from './components/GroupCom.vue';
import SelectCom from './components/SelectCom.vue';
import TableCom from './components/TableCom.vue';

const API_URL = import.meta.env.VITE_API_URL;

const data = ref([]);
const ITEMS_PER_PAGE = 30;
const page = ref(1);
const filteredData = ref([]);
const groups = ref(['ALL', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10']);
const activeGroup = ref('');
const completedUpload = ref(0);
const cancelCount = ref(0);

// 計算 完成檔案上傳筆數/取消報名筆數
const counter = () => {
  data.value.forEach((item) => {
    if (item.img_upload === 'y' && item.audio_upload === 'y') {
      completedUpload.value++;
    }

    if (item.cancel_sign_up === 'y') {
      cancelCount.value++;
    }
  });
};

// 選擇組別
const chooseGroup = (group) => {
  activeGroup.value = group;
  page.value = 1;

  if (group === 'ALL') {
    filteredData.value = data.value;
  } else {
    filteredData.value = data.value.filter((item) => item.group === group);
  }
};

// 計算總頁數
const totalPageNumber = computed(() => {
  return Math.ceil(filteredData.value.length / ITEMS_PER_PAGE);
});

// 產生分頁資料
const pageData = computed(() => {
  // 沒有搜尋字串
  if (searchString.value.trim().length === 0) {
    return filteredData.value.slice((page.value - 1) * ITEMS_PER_PAGE, page.value * ITEMS_PER_PAGE);
  }

  return filteredData.value;
});

// 繪製 C3 圖表
const drawChart = (data) => {
  const columns = [['各組別報名筆數 (包含取消報名)']];

  for (let i = 1; i <= 10; i++) {
    columns[0].push(data.filter((item) => item.group == i).length);
  }

  const chart = c3.generate({
    data: {
      columns,
      type: 'bar'
    },

    axis: {
      x: {
        type: 'category',
        categories: [
          '1. 年輕活力 A',
          '2. 年輕活力 B',
          '3. 創作 A',
          '4. 創作 B',
          '5. 經典 A',
          '6. 經典 B',
          '7 .動感/嘻哈 A',
          '8. 動感/嘻哈 B',
          '9. 情歌 A',
          '10. 情歌 B'
        ]
      }
    }
  });
};

// 搜尋
const searchString = ref('');
const regexp = ref('');

const escapeSpecialCharacters = (searchString) => {
  return searchString.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
};

watch(searchString, (newValue) => {
  if (newValue.trim().length === 0) {
    // 沒有搜尋字串
    regexp.value = '';

    const tempPage = page.value;
    chooseGroup(activeGroup.value);
    page.value = tempPage;
  } else {
    // 有搜尋字串
    regexp.value = new RegExp(escapeSpecialCharacters(newValue), 'gi');
    filteredData.value = data.value.filter((item) => item.nickname.match(regexp.value));
  }
});

onMounted(async () => {
  const res = await fetch(`${API_URL}/data.php`);
  data.value = await res.json();

  counter();
  chooseGroup('ALL');
  drawChart(data.value);
});

const nicknameForMsg = ref('');

// 點選 照片/音檔 按鈕
const uploadOkBtn = (item, type) => {
  if (item[`${type}_upload`] === 'y') {
    item[`${type}_upload`] = '';
  } else {
    item[`${type}_upload`] = 'y';
  }

  completedUpload.value = 0;
  cancelCount.value = 0;

  counter();

  const form = document.querySelector(`#form-${type}`);

  form.querySelector('input[name="user_no"]').value = item.no;
  form.querySelector(`input[name="${type}_status"]`).value = item[`${type}_upload`];

  const formData = new FormData(form);

  fetch(`${API_URL}/img-audio-db.php`, { method: 'POST', body: formData })
    .then((res) => res.json())
    .then(() => {
      nicknameForMsg.value = item.nickname;

      document.querySelector(`.${type}-msg`).classList.add('active');

      setTimeout(() => {
        document.querySelector(`.${type}-msg`).classList.remove('active');
      }, 3000);
    })
    .catch((err) => console.error(err));
};

// 點選 取消 按鈕
const cancelBtn = (item) => {
  if (window.confirm('確定要"取消報名"？ \n') === false) {
    return;
  }

  item.cancel_sign_up = 'y';

  completedUpload.value = 0;
  cancelCount.value = 0;

  counter();

  const formCancel = document.querySelector('#form-cancel');

  formCancel.querySelector('input[name="user_no"]').value = item.no;
  formCancel.querySelector('input[name="cancel_status"]').value = item.cancel_sign_up;

  const formData = new FormData(formCancel);

  fetch(`${API_URL}/cancel-db.php`, { method: 'POST', body: formData })
    .then((res) => res.json())
    .then(() => {
      nicknameForMsg.value = item.nickname;

      document.querySelector('.cancel-msg').classList.add('active');

      setTimeout(() => {
        document.querySelector('.cancel-msg').classList.remove('active');
      }, 3000);
    })
    .catch((err) => console.error(err));
};

// 修改 暱稱
const changeNickname = (item) => {
  const formUpdate = document.querySelector('#form-update');

  formUpdate.querySelector('input[name="user_no"]').value = item.no;
  formUpdate.querySelector('input[name="nickname"]').value = item.nickname;

  const formData = new FormData(formUpdate);

  fetch(`${API_URL}/update-db.php`, { method: 'POST', body: formData })
    .then((res) => res.json())
    .then(() => {
      document.querySelector('.update-msg').classList.add('active');

      setTimeout(() => {
        document.querySelector('.update-msg').classList.remove('active');
      }, 3000);
    })
    .catch((err) => console.error(err));
};

const removeActive = (e) => {
  e.target.classList.remove('active');
};
</script>

<template>
  <div class="row mb-4">
    <div class="col-3">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">總報名筆數 (包含取消報名)</h5>
          <p class="card-text fs-1 mb-0 lh-1 mt-3">{{ data.length }}</p>
        </div>
      </div>
    </div>
    <div class="col-3">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">完成檔案上傳筆數</h5>
          <p class="card-text fs-1 mb-0 lh-1 mt-3">{{ completedUpload }}</p>
        </div>
      </div>
    </div>
    <div class="col-3">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">取消報名筆數</h5>
          <p id="cancel-num" class="card-text fs-1 mb-0 lh-1 mt-3">{{ cancelCount }}</p>
        </div>
      </div>
    </div>
  </div>

  <div id="chart" class="mb-4 mx-auto"></div>

  <search-com v-model="searchString"></search-com>

  <template v-if="searchString.trim().length === 0">
    <group-com :groups="groups" :active-group="activeGroup" @choose-group="chooseGroup"></group-com>

    <select-com v-model="page" :total-page-number="totalPageNumber"></select-com>

    <p class="text-end">
      本組別共有 <b>{{ filteredData.length }}</b> 筆資料
    </p>
  </template>
  <template v-else>
    <p class="text-end">
      搜尋結果：<b>{{ filteredData.length }}</b> 筆資料
    </p>
  </template>

  <table-com
    :data="pageData"
    :regexp="regexp"
    @upload-ok-btn="uploadOkBtn"
    @cancel-btn="cancelBtn"
    @change-nickname="changeNickname"
  ></table-com>

  <p
    class="img-msg bg-success text-white py-3 px-2 w-25 text-center fw-bold fs-4"
    @click="removeActive"
  >
    <i class="fa fa-check-circle" aria-hidden="true"></i> {{ nicknameForMsg }} 照片狀態已修改
  </p>
  <p
    class="audio-msg bg-info text-dark py-3 px-2 w-25 text-center fw-bold fs-4"
    @click="removeActive"
  >
    <i class="fa fa-check-circle" aria-hidden="true"></i> {{ nicknameForMsg }} 音檔狀態已修改
  </p>
  <p
    class="cancel-msg bg-danger text-white py-3 px-2 w-25 text-center fw-bold fs-4"
    @click="removeActive"
  >
    <i class="fa fa-check-circle" aria-hidden="true"></i> {{ nicknameForMsg }} 已取消報名
  </p>
  <p
    class="update-msg bg-warning text-dark py-3 px-2 w-25 text-center fw-bold fs-4"
    @click="removeActive"
  >
    <i class="fa fa-check-circle" aria-hidden="true"></i> 暱稱已修改
  </p>
</template>
