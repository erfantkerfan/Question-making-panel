<template>
  <q-card class="profile-card">
    <q-img src="https://nodes.alaatv.com/upload/images/profile/default_avatar.jpg"
           width="300px"
           height="300px"
           dark
    />
    <q-card-actions vertical>
      <q-btn class="profile-link bg-green"
             :to="{ name: 'User.Exam.List' }"
      >
        بازگشت به پیشخوان
      </q-btn>
    </q-card-actions>
  </q-card>
</template>

<script>
import { mixinAuth, mixinQuiz } from 'src/mixin/Mixins'

export default {
  name: 'onlineQuizTopMenu',
  mixins: [mixinAuth, mixinQuiz],
  methods: {
    goHome () {
      this.$router.push({ name: 'User.Exam.List' })
    },
    sendAnswersAndFinishExam () {
      const that = this
      this.quiz.sendAnswersAndFinishExam()
        .then(() => {
          that.$store.commit('clearExamData', that.quiz.user_exam_id)
          that.$q.notify({
            message: 'اطلاعات آزمون شما ثبت شد.',
            type: 'positive'
          })
          that.$router.push({ name: 'User.Exam.List' })
        })
        .catch(() => {
          that.$q.notify({
            title: 'توجه!',
            message: 'مشکلی در ثبت اطلاعات آزمون شما رخ داده است. لطفا تا قبل از ساعت 24 اقدام به ارسال مجدد پاسخنامه نمایید.',
            type: 'warn',
            duration: 30000
          })
          that.$router.push({ name: 'User.Exam.List' })
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.profile-card {
  width: 100%;
  max-width: 324px;
  border-radius: 20px;
  .q-card__actions {
    padding: 0 !important;
    .q-btn {
      text-decoration: none;
      cursor: pointer;
      height: 44px;
      color: white;
      text-align: center;
      box-sizing: border-box;
      border-radius: 0 0 5px 5px;
    }
  }
}
</style>
