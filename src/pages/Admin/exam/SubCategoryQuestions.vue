<template>
  <div class="main-container">
    <q-card class="top-menu q-pa-lg">
      <div class="row items-center">
        <div class="col-8">
          <q-btn-toggle v-if="false"
                        v-model="questionFilterMethod"
                        unelevated
                        no-caps
                        toggle-color="primary"
                        color="white"
                        text-color="black"
                        :options="[
                          {label: 'نمایش همه', value:'not-filtered'},
                          {label: ' کلا تایید نشده', value:'not-confirmed-at-all'},
                          {label: 'من تایید نکردم', value:'not-confirmed-by-me'}
                        ]"
          />
          <q-btn v-if="false"
                 round
                 color="primary"
                 unelevated
                 icon="isax:printer"
                 @click="printQuestions"
          />
        </div>
        <div class="col-4 flex justify-end">
          <div class="search-box q-pr-md">
            <div>
              <q-input
                v-model="searchedQuestionOrder"
                type="number"
                outlined
                dense
                label="شماره سوال"
                @keydown.enter="paginateToQuestion"
              >
                <template v-slot:append>
                  <div @click="paginateToQuestion">
                    <i class="fi fi-rr-search search-icon cursor-pointer"></i>
                  </div>
                </template>
              </q-input>
            </div>
            <i class="fi fi-rr-refresh refresh-icon cursor-pointer q-ml-md"
               @click="reload"
            />
          </div>
          <q-btn
            round
            color="primary"
            unelevated
            @click="this.$router.go(-1)"
          >
            <i class="fi-rr-angle-left row" />
          </q-btn>
        </div>
      </div>
    </q-card>
    <div  v-if="pageLoading"
          class="text-center ">
      <q-spinner-ball
        class="q-my-xl"
        color="primary"
        size="5em"
      />
    </div>
    <template v-else>
      <q-virtual-scroll
        ref="scroller"
        :key="questionListKey"
        class="konkoor-view-scroll q-pa-md q-mt-md"
        :items="filteredQuestions"
        @virtual-scroll="onScroll"
      >
        <template v-slot="{ item, index }">
          <q-item :key="index"
                  class="question-field no-padding q-mb-md"
                  dense
          >
            <q-item-section class="no-wrap">
              <!--            :sub-category="quizData.sub_categories"-->
              <!--            :exam-id="$route.params.quizId"-->
              <!--            :consider-active-category="false"-->
              <!--            :questions-column="$refs.questionsColumn"-->
              <question-item pageStrategy="lesson-detail"
                             :question="item"
                             :confirmLoading="confirmQLoading"
                             :listOptions="questionsOptions"
                             :final-approval-mode="false"
                             :show-question-number="true"
                             @deleteFromExam="detachQuestion"
                             @deleteFromDb="deleteQuestion"
                             @copyIdToClipboard="copyIdToClipboard"
                             @confirmQuestion="confirmQuestion"
              />
            </q-item-section>
          </q-item>
        </template>
      </q-virtual-scroll>
    </template>
    <div class="pagination">
      <pagination
        :key="paginationKey"
        :meta="paginationMeta"
        :disable="pageLoading"
        @updateCurrentPage="updateQuizData"
      />
    </div>

  </div>
</template>

<script>
import Question from 'src/components/QuizEditor/Question'
import { mixinAuth, mixinQuiz } from 'src/mixin/Mixins'
import { QuestSubcategoryList } from 'src/models/QuestSubcategory'
import { QuestionList } from 'src/models/Question'
import { Exam } from 'src/models/Exam'
import API_ADDRESS from 'src/api/Addresses'
import QuestionItem from 'components/Question/QuestionItem/QuestionItem'
import { copyToClipboard } from 'quasar'
import Pagination from 'components/Question/QuestionBank/Pagination'
export default {
  name: 'SubCategoryQuestions',
  components: {
    Pagination,
    QuestionItem
  },
  mixins: [mixinAuth, mixinQuiz],
  data () {
    return {
      pageLoading: false,
      confirmQLoading: false,
      questionsOptions: {
        copy: true,
        detachQuestion: true,
        deleteQuestionFromDb: true,
        deleteQuestionFromExam: true,
        editQuestion: true,
        switch: true,
        questionAnswerExpanded: true
      },
      questionListKey: 0,
      options: {
        bookmark: false,
        copy: false,
        deleteQuestion: false,
        deleteQuestionFromDb: false,
        editQuestion: false,
        switch: false
      },
      subCategoriesList: new QuestSubcategoryList(),
      questionFilterMethod: 'not-filtered',
      quizData: new Exam(),
      item: Question,
      scrollState: 'not scrolling',
      timePassedSinceLastScroll: 0,
      setIntervalCallback: null,
      renderedQuestions: { startIndex: 0, endIndex: 0 },
      dialog: false,
      items: [
        'خارج از بودجه بندی آزمون ',
        'خارج از چارچوب کتاب درسی',
        'دارای غلط تایپی'
      ],
      inView: [],
      questionSearchNumber: 0,
      searchedQuestionOrder: 0,
      firstQuestionOrder: 0,
      disablePagination: false,
      paginationMeta: {
        current_page: 1,
        from: 0,
        last_page: 1,
        links: [],
        path: '',
        per_page: 0,
        to: 0,
        total: 0
      },
      paginationKey: 0
    }
  },
  created () {
    this.loadQuizDataAndSubCategories(false, this.$route.params.page)
  },
  computed: {
    filteredQuestions () {
      this.quizData.questions.list.forEach((item, index) => {
        item.questNumber = index + 1
      })
      let newList
      if (this.questionFilterMethod === 'not-confirmed-at-all') {
        newList = this.quizData.questions.list.filter(item => item.confirmers.length === 0)
      } else if (this.questionFilterMethod === 'not-confirmed-by-me') {
        newList = this.quizData.questions.list.filter(item => item.confirmed === false)
      } else {
        newList = this.quizData.questions.list
      }
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      this.questionListKey = Date.now()
      return newList
    },
    examId () {
      return this.$route.params.exam_id
    }
  },
  methods: {
    printQuestions () {
      const routeData = this.$router.resolve({ name: 'Admin.Exam.Lessons.PrintQuestions', params: { quizId: this.$route.params.exam_id, lessonId: this.$route.params.subcategory_id } })
      window.open(routeData.href, '_blank')
    },
    copyIdToClipboard (data) {
      copyToClipboard(data)
    },
    confirmQuestion (question) {
      this.confirmLoading = true
      question.confirmed ? this.unConfirmUser(question) : this.confirmUser(question)
    },
    async confirmUser (question) {
      try {
        const response = await this.sendConfirmReq(question)
        question.confirmed = response.data.data.confirmed
        question.confirmers = response.data.data.confirmers
        this.confirmLoading = false
      } catch (e) {
        question.confirmed = !question.confirmed
        this.confirmLoading = false
      }
    },
    async unConfirmUser (question) {
      try {
        const response = await this.sendUnConfirmReq(question)
        question.confirmed = response.data.data.confirmed
        question.confirmers = response.data.data.confirmers
        this.confirmLoading = false
      } catch (e) {
        question.confirmed = !question.confirmed
        this.confirmLoading = false
      }
    },
    sendUnConfirmReq (question) {
      return this.$axios.get(API_ADDRESS.question.unconfirm(question.id))
    },
    sendConfirmReq (question) {
      return this.$axios.get(API_ADDRESS.question.confirm(question.id))
    },
    paginateToQuestion () {
      if (this.paginationMeta.per_page === 0) {
        return
      }
      const page = Math.floor(this.searchedQuestionOrder / this.paginationMeta.per_page) + 1
      this.loadQuizDataAndSubCategories(false, page, () => {
        this.$nextTick(() => {
          const questionIndex = this.filteredQuestions.findIndex(question => question.in_subcategory_order.toString() === this.searchedQuestionOrder.toString())
          if (questionIndex === -1) {
            this.searchedQuestionOrder = 0
            this.$q.notify({
              message: 'سوال پیدا نشد.',
              type: 'negative'
            })
            return
          }
          this.$refs.scroller.scrollTo(questionIndex, 'start-force')
        })
      })
    },
    questionListHeight () {
      // box is a col-7 with 12px padding
      const boxSize = this.$refs.bubbleSheet.clientWidth - 24
      // each group width is 140px
      const horizontalGroupAmounts = Math.floor(boxSize / 140)
      const verticalGroupAmount = Math.ceil(this.questionsInGroups.length / horizontalGroupAmounts)
      return verticalGroupAmount * 182 + 20
    },
    changeAppBarAndDrawer (state) {
      // this.$store.commit('AppLayout/updateAppBarAndDrawer', state)
    },
    async loadQuizDataAndSubCategories (reload = false, pageNumber = 1, callback) {
      this.pageLoading = true
      if (this.$route.params.page !== pageNumber.toString()) {
        this.$router.push({
          name: 'Admin.Exam.SubCategory.Questions',
          params: {
            page: pageNumber,
            exam_id: this.$route.params.exam_id,
            subcategory_id: this.$route.params.subcategory_id
          }
        })
      }
      try {
        const response = await this.getQuizDataAndSubCategories(pageNumber)
        if (response.data.data.length) {
          this.paginationMeta = response.data.meta
          // this.paginationKey++
          this.firstQuestionOrder = response.data.data[0].order
          this.loadSubCategories(response, reload, callback)
          this.pageLoading = false
        } else {
          // this.$router.push({ name: 'Admin.Exam.Index' })
          this.$q.notify({
            type: 'negative',
            message: 'دیتای مورد نظر یافت نشد'
          })
        }
        this.pageLoading = false
      } catch (e) {
        console.error('err ', e)
        this.pageLoading = false
      }
    },
    getQuizDataAndSubCategories (pageNumber) {
      return this.$axios.post(API_ADDRESS.exam.examQuestion(this.examId, pageNumber), {
        sub_categories: [this.$route.params.subcategory_id]
      })
    },
    loadSubCategories (quizResponse, reload, callback) {
      const that = this
      this.$axios.get(API_ADDRESS.questionSubcategory.base)
        .then((response) => {
          if (reload) {
            that.$q.notify({
              message: 'اطلاعات بروزرسانی شد.',
              type: 'positive'
            })
          }
          that.quizData.sub_categories = new QuestSubcategoryList(response.data.data)
          const questions = quizResponse.data.data
          that.sortQuestions(questions)
          that.quizData.questions = new QuestionList(questions)
          this.questionListKey = Date.now()
          if (typeof callback === 'function') {
            callback(response)
          }
        })
    },
    scrollTo (questionId, questionNumber) {
      let questionIndex
      if (questionId) {
        questionIndex = this.getQuestionIndexById(questionId)
      } else {
        // questionIndex = questionNumber - this.firstQuestionOrder
        questionIndex = questionNumber
      }
      this.$refs.scroller.scrollTo(questionIndex, 'start-force')
      for (let i = 1; i < 4; i++) {
        setTimeout(() => {
          this.$refs.scroller.scrollTo(questionIndex)
        },
        333 * i)
      }
    },
    choiceClicked (questionId) {
      this.scrollTo(questionId)
      // this.changeQuestion(questionId)
    },
    reload () {
      this.loadQuizDataAndSubCategories(true, this.$route.params.page)
    },
    onScroll (startIndex, endIndex) {
      // this.updateLtr()
      this.renderedQuestions = { startIndex, endIndex }
      if (this.scrollState === 'not scrolling') {
        this.setIntervalCallback = setInterval(() => {
          this.changeCurrentQuestionIfScrollingIsDone()
        }, 250)
        this.scrollState = 'scrolling'
      }
      this.timePassedSinceLastScroll = 0
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
    changeCurrentQuestionToFirstQuestionInView () {
      const firstInViewQuestion = this.getFirstInViewQuestionNumber()
      if (firstInViewQuestion.id === this.currentQuestion.id) {
        return
      }
      this.changeQuestion(firstInViewQuestion.id, 'onlineQuiz.konkoorView')
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
    deleteQuestion (question) {
      this.$store.dispatch('AppLayout/showConfirmDialog', {
        show: true,
        message: 'از حذف کامل سوال از پایگاه داد و حذف از تمامی آزمون ها اطمینان دارید؟',
        buttons: {
          no: 'خیر',
          yes: 'بله'
        },
        callback: async (confirm) => {
          if (!confirm) {
            this.closeConfirmModal()
            return
          }
          try {
            this.closeConfirmModal()
            await this.deleteQuestionReq(question.id)
            this.$q.notify({
              message: 'سوال از پایگاه داده حذف شد.',
              type: 'positive'
            })
            this.reload()
          } catch (e) {
            this.reload()
            this.closeConfirmModal()
          }
        }
      })
    },
    deleteQuestionReq (questionId) {
      return this.$axios.delete(API_ADDRESS.question.delete(questionId))
    },
    detachQuestion (question) {
      this.$store.dispatch('AppLayout/showConfirmDialog', {
        show: true,
        message: 'از حذف سوال از آزمون اطمینان دارید؟',
        buttons: {
          no: 'خیر',
          yes: 'بله'
        },
        callback: async (confirm) => {
          if (!confirm) {
            this.closeConfirmModal()
          } else {
            try {
              this.closeConfirmModal()
              await this.detachQuestionReq(question.id)
              this.$q.notify({
                message: 'سوال از آزمون حذف شد.',
                type: 'positive'
              })
              this.reload()
            } catch (e) {
              this.reload()
              this.closeConfirmModal()
            }
          }
        }
      })
    },
    detachQuestionReq (questionId) {
      return this.$axios.post(API_ADDRESS.question.detach(questionId), {
        detaches: [{
          exam_id: this.examId,
          sub_category_id: this.$route.params.subcategory_id
        }]
      })
    },
    closeConfirmModal () {
      this.$store.commit('AppLayout/showConfirmDialog', {
        show: false
      })
    },
    updateQuizData (page) {
      this.loadQuizDataAndSubCategories(false, page)
    }
  }
}
</script>

<style lang="scss" scoped>
.search-box{
  display: flex;
  align-items: center;
  .search-icon{
    display: flex;
  }
  .refresh-icon{
    display: flex;
    font-size: 25px;
    color: #7b7b7b;
  }
}
.top-menu{
  border-radius: 15px;
  box-shadow: -2px -4px 10px rgba(255, 255, 255, 0.6), 2px 4px 10px rgba(54, 90, 145, 0.05);
}
.main-container{
  padding : 10px 40px;
  @media only screen and (max-width: 1919px){
    padding : 10px 30px;
  }
  @media only screen and (max-width: 599px){
    padding : 10px 16px;
  }
}
.konkoor-view-scroll {
  max-height: calc(100vh - 323px);
}
</style>
