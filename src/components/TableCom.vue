<script setup>
import { ref, nextTick } from 'vue'

defineProps(['data', 'regexp'])
const emit = defineEmits(['uploadOkBtn', 'cancelBtn', 'changeNickname'])

const API_URL = import.meta.env.VITE_API_URL

// 點選 照片/音檔 按鈕
const uploadOkBtn = (item, type) => {
  emit('uploadOkBtn', item, type)
}

// 點選 取消 按鈕
const cancelBtn = (item) => {
  emit('cancelBtn', item)
}

// 顯示詳細資料
const detailData = ref('')

const showDetail = (user) => {
  fetch(`${API_URL}/detail-db.php?u=${user}`)
    .then((res) => res.json())
    .then((data) => (detailData.value = data[0]))
    .catch((err) => console.log(err))
}

// 雙擊 暱稱欄位
const showNicknameField = ref(true)
const showInputNo = ref('')
let tempNickname
let changeNicknameStatus = false

const nicknameDoubleClick = (item) => {
  // 報名取消 不能 雙擊 暱稱欄位
  if (item.cancel_sign_up === 'y') {
    return
  }

  showNicknameField.value = false
  showInputNo.value = item.no
  tempNickname = item.nickname
  changeNicknameStatus = false

  nextTick(() => {
    const input = document.querySelector(`#input-${item.no}`)

    input.focus()

    input.addEventListener('blur', () => {
      // 按 enter 成功修改後 不會執行這個 if
      if (!changeNicknameStatus) {
        esc(item)
      }
    })
  })
}

// 按 enter 修改
const changeNickname = (item) => {
  const re = new RegExp(/["^{}()|[\]\\]/, 'i')

  // 暱稱 不能為空值 / 不能有特殊字元 / 修改前後不能相同
  if (!item.nickname.trim().length || re.test(item.nickname) || tempNickname === item.nickname) {
    if (item.nickname.trim().length < 1) {
      alert('請輸入暱稱')
    }

    if (re.test(item.nickname)) {
      alert('暱稱請勿使用特殊字元')
    }

    if (tempNickname === item.nickname) {
      alert('暱稱修改前後不能相同')
    }
  } else {
    changeNicknameStatus = true
    showNicknameField.value = true

    emit('changeNickname', item)
  }
}

// 按 esc 放棄修改
const esc = (item) => {
  item.nickname = tempNickname
  showNicknameField.value = true
}
</script>

<template>
  <table class="table table-bordered table-hover">
    <thead>
      <tr class="text-center">
        <th width="50">ID <i class="fa fa-caret-down" aria-hidden="true"></i></th>
        <th>姓名</th>
        <th>暱稱</th>
        <th>組別</th>
        <th>歌手</th>
        <th>歌名</th>
        <th>Email</th>
        <th>照片</th>
        <th>音檔</th>
        <th width="120"></th>
        <th width="120"></th>
        <th width="120"></th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="item in data"
        :key="item.no"
        :class="{ 'text-muted': item.cancel_sign_up === 'y' }"
      >
        <td class="position-relative">
          <a
            href=""
            class="block-link text-decoration-none position-absolute p-2"
            data-bs-toggle="modal"
            data-bs-target="#detailModal"
            @click="showDetail(item.no)"
            >{{ item.no }}</a
          >
          <i
            class="fa fa-times-circle text-danger"
            aria-hidden="true"
            v-if="item.cancel_sign_up === 'y'"
          ></i>
        </td>
        <td>{{ item.name }}</td>
        <td v-if="showNicknameField" class="highlight" @dblclick="nicknameDoubleClick(item)">
          <div
            class="d-inline-block"
            v-html="
              item.nickname.replace(regexp, (match) => {
                if (match !== '') {
                  return `<span>${match}</span>`
                }

                return match
              })
            "
          ></div>
          <div class="edit-icon d-inline-block" v-if="item.cancel_sign_up !== 'y'"></div>
        </td>
        <td v-else-if="item.no === showInputNo">
          <input
            :id="`input-${item.no}`"
            type="text"
            v-model.trim="item.nickname"
            @keyup.enter="changeNickname(item)"
            @keyup.esc="esc(item)"
          />
        </td>
        <td v-else>{{ item.nickname }}</td>
        <td>{{ item.group }}</td>
        <td v-html="item.singer"></td>
        <td v-html="item.song"></td>
        <td>{{ item.email }}</td>
        <td class="text-center text-success">
          <template v-if="item.img_upload === 'y'">
            <i class="fa fa-check-circle" aria-hidden="true"></i>
          </template>
        </td>
        <td class="text-center text-info">
          <template v-if="item.audio_upload === 'y'">
            <i class="fa fa-check-circle" aria-hidden="true"></i>
          </template>
        </td>
        <td>
          <button
            type="button"
            class="btn btn-success w-100"
            :disabled="item.cancel_sign_up === 'y'"
            @click="uploadOkBtn(item, 'img')"
          >
            照片
          </button>
        </td>
        <td>
          <button
            type="button"
            class="btn btn-info w-100"
            :disabled="item.cancel_sign_up === 'y'"
            @click="uploadOkBtn(item, 'audio')"
          >
            音檔
          </button>
        </td>
        <td>
          <button
            type="button"
            class="btn btn-danger w-100"
            :disabled="item.cancel_sign_up === 'y'"
            @click="cancelBtn(item)"
          >
            取消
          </button>
        </td>
      </tr>
    </tbody>
  </table>

  <!-- form-img -->
  <form id="form-img">
    <input type="hidden" name="user_no" value="" />
    <input type="hidden" name="img_status" value="" />
  </form>

  <!-- form-audio -->
  <form id="form-audio">
    <input type="hidden" name="user_no" value="" />
    <input type="hidden" name="audio_status" value="" />
  </form>

  <!-- form-cancel -->
  <form id="form-cancel">
    <input type="hidden" name="user_no" value="" />
    <input type="hidden" name="cancel_status" value="" />
  </form>

  <!-- form-update -->
  <form id="form-update">
    <input type="hidden" name="user_no" value="" />
    <input type="hidden" name="nickname" value="" />
  </form>

  <!-- Modal -->
  <div
    class="modal fade"
    id="detailModal"
    tabindex="-1"
    aria-labelledby="detailModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="detailModalLabel"></h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <table class="table table-bordered">
            <tbody>
              <tr>
                <th scope="row" width="220">ID</th>
                <td>{{ detailData.no }}</td>
              </tr>
              <tr>
                <th scope="row">Name</th>
                <td>{{ detailData.name }}</td>
              </tr>
              <tr>
                <th scope="row">Nickname</th>
                <td>{{ detailData.nickname }}</td>
              </tr>
              <tr>
                <th scope="row">Group</th>
                <td>{{ detailData.group }}</td>
              </tr>
              <tr>
                <th scope="row">Singer / Song</th>
                <td>
                  <span v-html="detailData.singer"></span> /
                  <span v-html="detailData.song"></span>
                </td>
              </tr>
              <tr>
                <th scope="row">Gender</th>
                <td>{{ detailData.gender }}</td>
              </tr>
              <tr>
                <th scope="row">Birth</th>
                <td>{{ detailData.birth }}</td>
              </tr>
              <tr>
                <th scope="row">Phone</th>
                <td>{{ detailData.phone }}</td>
              </tr>
              <tr>
                <th scope="row">Email</th>
                <td>{{ detailData.email }}</td>
              </tr>
              <tr>
                <th scope="row">Nationality</th>
                <td>{{ detailData.nationality }}</td>
              </tr>
              <tr>
                <th scope="row">City</th>
                <td>{{ detailData.city }}</td>
              </tr>
              <tr>
                <th scope="row">Occupation</th>
                <td>{{ detailData.occupation }}</td>
              </tr>
              <tr>
                <th scope="row">Know</th>
                <td>{{ detailData.know }}</td>
              </tr>
              <tr>
                <th scope="row">照片</th>
                <td>
                  <template v-if="detailData.img_upload === 'y'">
                    <i class="fa fa-check-circle text-success" aria-hidden="true"></i> OK
                  </template>
                </td>
              </tr>
              <tr>
                <th scope="row">音檔</th>
                <td>
                  <template v-if="detailData.audio_upload === 'y'">
                    <i class="fa fa-check-circle text-info" aria-hidden="true"></i> OK
                  </template>
                </td>
              </tr>
              <tr>
                <th scope="row">取消報名</th>
                <td>
                  <template v-if="detailData.cancel_sign_up === 'y'">
                    <i class="fa fa-check-circle text-danger" aria-hidden="true"></i> 已取消報名
                  </template>
                </td>
              </tr>
              <tr>
                <th scope="row">Guardian Name</th>
                <td>{{ detailData.guardian_name }}</td>
              </tr>
              <tr>
                <th scope="row">Guardian Phone</th>
                <td>{{ detailData.guardian_phone }}</td>
              </tr>
              <tr>
                <th scope="row">Date</th>
                <td>{{ detailData.insertDate }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>
