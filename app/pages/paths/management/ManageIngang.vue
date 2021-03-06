<script>
import ContentWrapper from '@/components/ContentWrapper.vue'

import days from '@/src/util/days'
import { ingangManager } from '@/src/api/ingang'

export default {
  name: 'ManageIngang',
  components: { ContentWrapper },
  data: () => ({
    pending: false,
    currentTab: 0,
    ingangs: [],
    users: [],
    applies: [],
    notice: {
      grade: '',
      desc: ''
    },

    form: {
      grade: 1,
      day: 0,
      time: 1,
      date: '',
      startDate: new Date(),
      endDate: new Date(),
      maxUser: {}
    },

    black: {
      serial: null,
      count: null,
      date: new Date()
    }
  }),

  computed: {
    days: () => days,
    userFirst () {
      return this.users.filter(v => v.time === 1)
    },
    userSecond () {
      return this.users.filter(v => v.time === 2)
    }
  },

  async created () {
    await this.refresh()
  },

  methods: {
    async refresh () {
      this.pending = true
      this.ingangs = await ingangManager.getIngangs()
      this.notice.desc = (await ingangManager.getAnnouncement()).description
      this.updateAppliers()
      this.pending = false
    },

    async updateAppliers () {
      this.applies = await ingangManager.getIngangAppliers()
      this.users = this.applies.sort((a, b) => a.grade - b.grade)
      this.users = this.users.filter((v, i) => {
        return i === this.users.findIndex(_v => v.serial === _v.serial)
      })
    },

    async addNotice () {
      try {
        await ingangManager.addAnnouncement({
          'description': this.notice.desc,
          'grade': this.notice.grade
        })
        await this.$swal('추가하였습니다', '', 'success')
        await this.refresh()
      } catch (err) {
        this.$swal('이런!', err.message, 'error')
      }
    },

    async addIngang () {
      try {
        await ingangManager.createIngang(this.form)

        this.$swal('성공!', '추가되었습니다.', 'success')
        await this.refresh()
      } catch (err) {
        this.$swal('이런!', err.message, 'error')
      }
    },

    async addBlacklist () {
      try {
        await ingangManager.createIngangBlack(this.black)
        this.black = {
          serial: null,
          count: null,
          date: new Date()
        }
        this.$swal('성공!', '추가되었습니다.', 'success')
      } catch (err) {
        this.$swal('이런!', err.message, 'error')
      }
    },

    async downloadExcel (grade) {
      try {
        await ingangManager.downloadExcel(grade)
      } catch (err) {
        this.$swal('이런!', err.message, 'error')
      }
    },

    async deleteIngang (idx) {
      await ingangManager.deleteIngang(idx)
      await this.refresh()
    },

    getDayText (code) {
      return days.find(v => v.code === code).text
    },

    userAppliedTimes (serial) {
      return this.applies
        .filter(v => v.serial === serial)
        .map(v => `${v.time}타임`)
        .sort()
    }
  }
}
</script>

<template>
  <content-wrapper>
    <h1 slot="header">
      <span class="icon-internet-class" />인강실 신청 관리
    </h1>
    <dimi-card
      slot="main"
      class="ingang"
    >
      <dimi-tab
        v-model="currentTab"
        :tabs="['엑셀', '공지', '인강실', '사용자']"
      />
      <div
        v-if="pending"
        class="ingang__loader"
      >
        <dimi-loader />
      </div>
      <template v-else>
        <div
          v-if="currentTab === 0"
          class="excel"
        >
          <dimi-button-group
            :items="['1학년 엑셀 다운', '2학년 엑셀 다운']"
            class="excel__item"
            @click="downloadExcel($event.value + 1)"
          />
        </div>
        <div
          v-if="currentTab === 1"
        >
          <div class="notice">
            <dimi-input
              v-model="notice.grade"
              class="notice__input"
              placeholder="공지사항을 보내고자 하는 학년을 입력해주세요."
            />
          </div>
          <div class="notice">
            <dimi-long-input
              v-model="notice.desc"
              :height="300"
              class="notice__input"
            />
          </div>
          <div class="notice__button">
            <dimi-button
              class="notice__button"
              @click="addNotice()"
            >
              공지 수정하기
            </dimi-button>
          </div>
        </div>
        <div
          v-if="currentTab === 2"
        >
          <section class="mng-ing__section">
            <table class="mng-ing__list">
              <tbody>
                <tr
                  v-for="(item, index) in ingangs"
                  :key="index"
                  class="mng-ing__row"
                >
                  <td class="mng-ing__cell">
                    {{ getDayText(item.day) }}
                  </td>
                  <td class="mng-ing__cell">
                    {{ item.grade }}학년 {{ item.klass }}반
                  </td>
                  <td class="mng-ing__cell mng-ing__cell--time">
                    {{ item.time }}타임
                  </td>
                  <td class="mng-ing__cell">
                    총 {{ item.max }}명
                  </td>
                  <td class="mng-ing__cell">
                    {{ item.count }}명 신청
                  </td>
                  <td class="mng-ing__cell">
                    <span
                      class="mng-ing__tool mng-ing__delete"
                      @click="deleteIngang(item.idx)"
                    >
                      <span class="mng-ing__delete-icon icon-delete" /> 삭제
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </section>

          <section class="mng-ing__section">
            <h2 class="mng-ing__title">
              인강실 추가
            </h2>
            <div class="mng-ing__form">
              <div class="mng-ing__form-row">
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    학년
                  </label>
                  <dimi-input
                    v-model="form.grade"
                    class="mng-ing__input"
                  />
                </div>

                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    타임
                  </label>
                  <dimi-input
                    v-model.number="form.time"
                    class="mng-ing__input"
                  />
                </div>

                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    날짜
                  </label>
                  <dimi-input
                    v-model="form.date"
                    type="date"
                  />
                </div>
              </div>

              <div class="mng-ing__form-row">
                <div
                  v-for="klass in 6"
                  :key="`ingang-${klass}`"
                  class="mng-ing__field"
                >
                  <label class="mng-ing__label">
                    {{ klass }}반 최대인원
                  </label>
                  <dimi-input
                    v-model.number="form.maxUser[klass]"
                    class="mng-ing__input"
                  />
                </div>
              </div>

              <div class="mng-ing__form-row">
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    신청 시작
                  </label>
                  <dimi-date-input v-model="form.startDate" />
                </div>
              </div>

              <div class="mng-ing__form-row">
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    신청 마감
                  </label>
                  <dimi-date-input v-model="form.endDate" />
                </div>
              </div>

              <div class="mng-ing__form-row mng-ing__form-row--submit">
                <div class="mng-ing__field mng-ing__field--right">
                  <dimi-button @click="addIngang">
                    추가하기
                  </dimi-button>
                </div>
              </div>
            </div>
          </section>
        </div>
        <div
          v-if="currentTab === 3"
        >
          <section class="mng-ing__section">
            <div class="mng-ing__list-wrap">
              <div class="mng-ing__list-section">
                <h2 class="mng-ing__list-title">
                  1타임 인강실 신청자 목록
                </h2>
                <table class="mng-ing__list">
                  <tbody>
                    <tr
                      v-for="(user, index) in userFirst"
                      :key="index"
                      class="mng-ing__row"
                    >
                      <td class="mng-ing__cell">
                        {{ `${user.grade}학년 ${user.klass}반 ${user.number}번 ${user.name}` }}
                      </td>
                      <td class="mng-ing__cell mng-ing__cell--time">
                        {{ userAppliedTimes(user.serial).join(', ') }}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div class="mng-ing__list-section">
                <h2 class="mng-ing__list-title">
                  2타임 인강실 신청자 목록
                </h2>
                <table class="mng-ing__list">
                  <tbody>
                    <tr
                      v-for="(user, index) in userSecond"
                      :key="index"
                      class="mng-ing__row"
                    >
                      <td class="mng-ing__cell">
                        {{ `${user.grade}학년 ${user.klass}반 ${user.number}번 ${user.name}` }}
                      </td>
                      <td class="mng-ing__cell mng-ing__cell--time">
                        {{ userAppliedTimes(user.serial).join(', ') }}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>
          <section class="mng-ing__section">
            <h2 class="mng-ing__title">
              블랙리스트 추가
            </h2>
            <div class="mng-ing__form">
              <div class="mng-ing__form-row">
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    학번
                  </label>
                  <dimi-input
                    v-model.number="black.serial"
                    class="mng-ing__input"
                  />
                </div>
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    적발 횟수
                  </label>
                  <dimi-input
                    v-model.number="black.count"
                    class="mng-ing__input"
                  />
                </div>
              </div>
              <div class="mng-ing__form-row">
                <div class="mng-ing__field">
                  <label class="mng-ing__label">
                    해제 날짜
                  </label>
                  <dimi-date-input
                    v-model="black.date"
                    class="mng-ing__input"
                  />
                </div>
              </div>
              <div class="mng-ing__form-row mng-ing__form-row--submit">
                <div class="mng-ing__field mng-ing__field--right">
                  <dimi-button @click="addBlacklist">
                    추가하기
                  </dimi-button>
                </div>
              </div>
            </div>
          </section>
        </div>
      </template>
    </dimi-card>
  </content-wrapper>
</template>

<style lang="scss" scoped>
.mng-ing {
  &__section {
    padding: 0 24px 24px;
    margin-top: 36px;
    margin-bottom: 24px;
  }

  &__section:last-child {
    padding-bottom: 0;
  }

  &__title {
    margin-bottom: 48px;
    color: $gray-dark;
    font-size: 24px;
    font-weight: $font-weight-bold;
  }

  &__delete {
    display: flex;
    align-items: center;
    cursor: pointer;
    user-select: none;
  }

  &__delete-icon {
    font-size: 18px;
  }

  &__list-wrap {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    padding-bottom: 1.5rem;
  }

  &__list-section {
    display: flex;
    width: 48%;
    flex-direction: column;
  }

  &__list-title {
    padding: 24px;
    padding-top: 0;
    color: $gray-dark;
    font-size: 24px;
    font-weight: $font-weight-bold;
  }

  &__list {
    display: block;
    width: 100%;
    height: 600px;
    color: $gray !important;
    font-weight: $font-weight-bold;
    overflow-y: auto;
  }

  &__list::-webkit-scrollbar {
    display: none;
  }

  &__row:not(:last-child) {
    border-bottom: 1px solid $gray-lighter;
  }

  &__cell {
    padding: 24px;
    white-space: nowrap;
  }

  &__cell--time {
    width: 99%;
    line-height: 1.5;
    white-space: normal;
  }

  &__form {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  &__form-row {
    display: flex;
    margin-bottom: 24px;
  }

  &__form-row--submit {
    margin-top: 10px;
    margin-bottom: 24px;
  }

  &__field {
    display: flex;
    align-items: center;
  }

  &__field--full {
    flex-grow: 1;
  }

  &__field--right {
    margin-left: auto;
  }

  &__label {
    width: 5em;
    max-width: 4em;
    padding-right: 1em;
    text-align: right;
  }

  &__input {
    font-size: 16px;
  }

  &__input--manager {
    width: 10em;
  }

  &__input--maxCount {
    width: 6em;
  }

  &__input--day {
    padding: 0.75em 1em;
    border: 0;
    background-color: $gray-lighten;
    border-radius: 20px;
  }
}

.ingang {
  padding: 0;
  border: 0;

  &__loader {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.excel {
  display: flex;
  justify-content: center;
  margin: 16px;

  &__item {
    margin: 16px;
  }
}

.notice {
  display: flex;
  justify-content: center;

  &__input {
    padding: 0 12px;
    margin: 6px 0;
  }

  &__button {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    margin: 3px 8px 6px 0;
  }
}
</style>
