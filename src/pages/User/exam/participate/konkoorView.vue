<template>
  <div class="konkoor-view row">
    <div
      id="questions"
      ref="questionsColumn"
      class="col-md-5 right-side"
    >
      <q-virtual-scroll
        ref="scroller"
        class="konkoor-view-scroll"
        :items="questions"
        :virtual-scroll-item-size="450"
        :virtual-scroll-slice-size="5"
        @virtual-scroll="onScroll"
      >
        <template v-slot="{ item, index }">
          <q-item
            :key="index"
            class="question-field"
            dense
          >
            <q-item-section>
              <Item
                :source="item"
                :questions-column="$refs.questionsColumn"
                @inView="isInView"
              />
            </q-item-section>
          </q-item>
        </template>
      </q-virtual-scroll>
    </div>
    <div class="left-side col-md-7">
      <div class="konkoor-view-navbar">
        <q-btn
          icon="mdi-table-split-cell"
          color="grey"
          flat
          fab-mini
          @click="changeView(getAlaaViewRouteName())"
        />
        <q-btn-dropdown
          class="dropdown-button"
          icon="account_circle"
          :label="user.full_name "
          color="grey-14"
          dropdown-icon="false"
          flat
        >
          <top-menu />
        </q-btn-dropdown>
      </div>
      <div
        class="bubbleSheet-warpper"
      >
        <BubbleSheet
          :info="{ type: 'pasokh-barg'}"
          :delay-time="0"
          :questions="questions"
          :bubble-sheet-height="windowSize.y"
          @clickChoice="choiceClicked"
          @scrollTo="scrollTo"
        />
      </div>
      <div class="row timer-row">
        <q-btn
          class="end-exam-btn"
          :loading="confirmationBtnLoading"
          :disabled="confirmationBtnLoading"
          @click="getConfirmation"
        >
          ارسال پاسخنامه
        </q-btn>
        <div
          class="col"
          :class="{ 'high-z-index': timerIsOpen}"
        >
          <Timer
            :daftarche="'عمومی'"
            :quiz-started-at="1607963897"
            :daftarche-end-time="1607999897"
            :height="100"
            @timerOpen="timerOpen"
          />
        </div>
      </div>
    </div>
    <q-dialog
      v-model="confirmationBubbleSheet"
      persistent
      maximized
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="">
        <q-bar class="bg-blue text-white q-pa-lg">
          <div>
            پاسخنامه کاربر
          </div>
          <q-space></q-space>
          <q-btn v-close-popup
                 dense
                 flat
                 icon="close">
            <q-tooltip class="bg-white text-blue">بستن</q-tooltip>
          </q-btn>
        </q-bar>
        <q-card-section>
          <q-card flat>
            <q-card-section>
              از ارسال پاسخ ها اطمینان دارید؟
            </q-card-section>
            <q-card-section>
              <q-btn
                flat
                style="color: #585858"
                @click="confirmationBubbleSheet = false"
              >
                ادامه میدم
              </q-btn>
              <q-btn
                flat
                color="secondary"
                @click="confirmSendingAllAnswers"
              >
                ثبت میکنم
              </q-btn>
            </q-card-section>
          </q-card>
        </q-card-section>

        <q-card-section>
          <bubble-sheet
            :info="{ type: 'pasokh-nameh' }"
            delay-time="0"
          />
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import Item from 'src/components/OnlineQuiz/Quiz/question/questionField'
import { mixinAuth, mixinQuiz, mixinUserActionOnQuestion } from 'src/mixin/Mixins'
import Timer from 'src/components/OnlineQuiz/Quiz/timer/timer'
import BubbleSheet from 'src/components/OnlineQuiz/Quiz/bubbleSheet/bubbleSheet'
import { Exam } from 'src/models/Exam'
import Assistant from 'src/plugins/assistant'
import TopMenu from 'src/components/Menu/topMenu/onlineQuizTopMenu'
import { ref } from 'vue'
import { mapGetters } from 'vuex'

import 'src/assets/scss/markdownKatex.scss'
import ExamData from 'assets/js/ExamData'

export default {
  name: 'konkoorView',
  components: {
    Timer,
    TopMenu,
    BubbleSheet,
    Item
  },
  mixins: [mixinAuth, mixinQuiz, mixinUserActionOnQuestion],
  data () {
    return {
      quizData: new Exam(),
      item: Item,
      lastTimeScrollRange: { start: 0, end: 29 },
      scrollState: 'not scrolling',
      timePassedSinceLastScroll: 0,
      setIntervalCallback: null,
      renderedQuestions: { startIndex: 0, endIndex: 0 },
      questions: [],
      inView: [],
      confirmationBubbleSheet: false,
      confirmationBtnLoading: false,
      timerIsOpen: false,
      scroller: ref(null),
      leftSideList: ref(null)
    }
  },
  watch: {
    'windowSize.x': function () {
      this.view()
    },
    'windowSize.y': function () {
      this.setHeights()
    }
  },
  mounted () {
    // if (this.$route.name === 'konkoorView') {
    //   this.changeAppBarAndDrawer(false)
    // }
    this.startExamProcess()
    this.setHeights()
    this.view()
    if (this.currentQuestion) {
      if (this.currentQuestion.id) {
        this.scrollTo(this.currentQuestion.id)
      } else {
        this.loadFirstQuestion()
      }
    }
  },
  unmounted () {
    this.changeAppBarAndDrawer(true)
  },
  computed: {
    ...mapGetters('AppLayout', [
      'windowSize'
    ])
  },
  methods: {
    startExamProcess () {
      const that = this
      const isPersonalExam = this.$route.name === 'onlineQuiz.konkoorView.personal'
      const retake = false

      this.startExam(this.$route.params.quizId, this.$route.name, retake, isPersonalExam)
        .then(() => {
          this.questions = this.getCurrentExamQuestionsInArray()
          const callbacks = {
            'question.file-link:update': {
              afterReload () {
                that.questions = that.getCurrentExamQuestionsInArray()
              }
            }
          }
          this.setSocket(this.$store.getters['Auth/accessToken'], this.quiz.id, callbacks)
          // this.updateOverlay(false)
        })
        .catch((error) => {
          Assistant.reportErrors(error)
          // this.updateOverlay(false)
          this.$router.push({ name: 'User.Exam.List' })
        })
    },
    timerOpen (value) {
      this.timerIsOpen = value
    },
    getConfirmation () {
      this.confirmationBtnLoading = true
      this.syncUserAnswersWithDBAndSendAnswersToServerInExamTime(this.quiz.user_exam_id, false)
        .then(() => {
          const examData = new ExamData()
          examData.getUserExamData(this.quiz.user_exam_id)
            .run()
            .then(() => {
              this.$store.commit('mergeDbAnswersIntoLocalstorage', {
                dbAnswers: examData.userExamData,
                exam_id: examData.exam.id
              })
              this.confirmationBubbleSheet = true
              this.confirmationBtnLoading = false
            })
            .catch(() => {
              this.confirmationBubbleSheet = true
              this.confirmationBtnLoading = false
            })
        })
        .catch(() => {
          this.confirmationBubbleSheet = true
          this.confirmationBtnLoading = false
        })
    },
    confirmSendingAllAnswers () {
      const isPersonalExam = this.$route.name === 'onlineQuiz.alaaView.personal' || this.$route.name === 'onlineQuiz.konkoorView.personal'
      const finishExam = isPersonalExam
      this.syncUserAnswersWithDBAndSendAnswersToServerInExamTime(this.quiz.user_exam_id, finishExam)
        .then(() => {
          this.$router.push({ name: 'User.Exam.List' })
          this.confirmationBubbleSheet = true
        })
        .catch(erroe => {
          console.error('erroe : ', erroe)
        })
    },
    sendAnswersAndFinishExam () {
      const isPersonalExam = this.$route.name === 'onlineQuiz.konkoorView.personal'
      const finishExam = isPersonalExam
      this.sendUserQuestionsDataToServerAndFinishExam(this.quiz.user_exam_id, finishExam)
        .then(() => {
          this.$store.commit('clearExamData', this.quiz.user_exam_id)
          this.$router.push({ name: 'User.Exam.List' })
        })
        .catch(() => {
          this.$q.notify({
            title: 'توجه!',
            message: 'مشکلی در ثبت اطلاعات آزمون شما رخ داده است. لطفا تا قبل از ساعت 24 اقدام به ارسال مجدد پاسخنامه نمایید.',
            type: 'warning',
            duration: 30000
          })
          this.$router.push({ name: 'User.Exam.List' })
        })
    },
    isInView (payload) {
      if (payload.isInView) {
        for (let i = 0; i < this.inView.length; i++) {
          if (this.inView[i] === payload.number) {
            return
          }
        }
        this.inView.push(payload.number)
      } else {
        for (let i = 0; i < this.inView.length; i++) {
          if (this.inView[i] === payload.number) {
            this.inView.splice(i, 1)
          }
        }
      }
    },
    changeAppBarAndDrawer (state) {
      this.$store.commit('AppLayout/updateLayoutHeader', state)
      this.$store.commit('AppLayout/updateLayoutLeftDrawer', state)
      this.$store.commit('AppLayout/updateLayoutRightDrawerVisible', state)
    },
    changeCurrentQuestionIfScrollingIsDone () {
      if (this.timePassedSinceLastScroll >= 1000) {
        this.changeCurrentQuestionToFirstQuestionInView()
        this.timePassedSinceLastScroll = 0
        this.scrollState = 'not scrolling'
        clearInterval(this.setIntervalCallback)
        this.setIntervalCallback = null
      }
      this.timePassedSinceLastScroll += 250
    },
    updateLtr () {
      setTimeout(() => {
        document.querySelectorAll('.katex:not([dir="ltr"])').forEach(item => {
          item.setAttribute('dir', 'ltr')
        })
      }, 1000)
    },
    onScroll (details) {
      if (!this.questions[details.index]) {
        return
      }
      this.changeQuestion(this.questions[details.index].id, this.$route.name)

      // startIndex, endIndex
      // this.updateLtr()
      // this.renderedQuestions = { startIndex, endIndex }
      // if (this.scrollState === 'not scrolling') {
      //   this.setIntervalCallback = setInterval(() => {
      //     this.changeCurrentQuestionIfScrollingIsDone()
      //   }, 250)
      //   this.scrollState = 'scrolling'
      // }
      // this.timePassedSinceLastScroll = 0
    },
    changeCurrentQuestionToFirstQuestionInView () {
      const firstInViewQuestion = this.getFirstInViewQuestionNumber()
      if (firstInViewQuestion.id === this.currentQuestion.id) {
        return
      }
      this.changeQuestion(firstInViewQuestion.id, this.$route.name)
    },
    scrollTo (questionId) {
      if (!this.$refs.scroller) {
        return
      }
      const questionIndex = this.getQuestionIndexById(questionId)
      this.$refs.scroller.scrollTo(questionIndex)
      for (let i = 1; i < 4; i++) {
        setTimeout(() => {
          if (this.$refs.scroller) {
            this.$refs.scroller.scrollTo(questionIndex)
          }
        },
        333 * i)
      }
    },
    getFirstInViewQuestionNumber () {
      let firstQuestionInView
      for (let i = this.renderedQuestions.startIndex; i <= this.renderedQuestions.endIndex; i++) {
        if (this.questions[i].isInView) {
          firstQuestionInView = this.questions[i]
          break
        }
      }
      if (firstQuestionInView && firstQuestionInView.id !== null) {
        return firstQuestionInView
      } else {
        return false
      }
    },
    choiceClicked (questionId) {
      this.scrollTo(questionId)
      this.changeQuestion(questionId, this.$route.name)
    },
    view () {
      if (this.windowSize.x > 1024) {
        this.changeAppBarAndDrawer(false)
      } else {
        this.$router.push({
          name: 'onlineQuiz.alaaView',
          params: { quizId: this.$route.params.quizId, questNumber: 1 }
        })
      }
    },
    getAlaaViewRouteName () {
      const isPersonalExam = this.$route.name === 'onlineQuiz.konkoorView.personal'
      const retake = this.$route.name === 'onlineQuiz.konkoorView.retake'

      if (isPersonalExam) {
        return 'onlineQuiz.alaaView.personal'
      }
      if (retake) {
        return 'onlineQuiz.alaaView.retake'
      }

      return 'onlineQuiz.alaaView'
    },
    setHeights () {
      if (this.$refs.scroller.$el) {
        this.$refs.scroller.$el.style.height = this.windowSize.y + 'px'
      }
      // this.$refs.leftSideList.style.height = (this.windowSize.y - 127) + 'px'
    },
    updateOverlay (value) {
      this.$store.dispatch('loading/overlayLoading', { loading: value })
    }
  }
}
</script>

<style lang="scss" scoped>
.konkoor-view {
  max-height: 100vh;
  //background-color: rgb(244, 244, 244);

  .right-side {
    height: 100%;
    display: flex;
    flex-direction: column;
    position: relative;
    padding: 0;

    .konkoor-view-scroll {
      max-height: 100%;

      .question-field {
        &.q-item {
          padding: 0;
        }
        &:deep(.reading-duplicate) {
          display: none;
        }
      }
    }
  }

  .left-side {
    .konkoor-view-navbar {
      display: flex;
      justify-content: space-between;
      padding: 0 40px;

      .dropdown-button {
        padding: 0;
        direction: rtl;
      }
    }

    .bubbleSheet-warpper {
      height: calc(100vh - 40px - 103px);
    }

    .timer-row {
      width: calc(58% - 150px);
      position: fixed;
      bottom: 0;
      right: 100px;

      .end-exam-btn {
        position: absolute;
        bottom: 0;
        background: rgb(76, 175, 80) !important;
        color: #fff;
        font-weight: bold;
        font-size: 16px;
        height: 103px !important;
        box-shadow: 0px 3px 3px -2px rgb(0 0 0 / 20%), 0px 3px 4px 0px rgb(0 0 0 / 14%), 0px 1px 8px 0px rgb(0 0 0 / 12%);
        width: 200px;
        border-radius: 20px 20px 0 0;
      }

      .high-z-index {
        z-index: 3;
      }
    }
  }

}
</style>
<style lang="scss">
.konkoor-view {
  .left-side {
    height: 100vh;
    .konkoor-view-navbar {
      .dropdown-button {
        .q-icon {
          font-size: 30px;

          &.on-left {
            margin: 0 0 0 8px;
          }
        }
      }

      .q-btn--fab-mini {
        padding: 0;
        height: 36px;
        width: 36px;
      }
    }
  }
}
</style>
