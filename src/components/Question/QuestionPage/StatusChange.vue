<template>
  <div class="question-details">
    <!--    <div class="box-title">مشخصات سوال</div>-->
    <div class="details-container-2 default-details-container row">
      <div class="detail-box detail-box-first"
           :class="[imgPanelVisibility ? 'col-6' : 'col-3']">
        <div class="detail-box-title">تعییر وضعیت</div>
        <q-select
          v-model="newStatus.changeState"
          borderless
          :options="statuses.list"
          option-value="id"
          option-label="display_title"
          :rules="selectorRules"
          :loading="statuses.loading"
        />
      </div>
      <div class="detail-box"
           :class="[imgPanelVisibility ? 'col-6' : 'col-3']">
        <div class="detail-box-title">افزودن کامنت</div>
        <q-input v-model="newStatus.commentAdded"
                 borderless
                 type="text" />
      </div>
      <div class="detail-box detail-box-last-of-row"
           :class="[imgPanelVisibility ? 'col-6' : 'col-3']">
        <q-btn
          unelevated
          :loading="statuses.loading"
          color="primary"
          label="ذخیره"
          class="save-btn default-detail-btn"
          @click="sendStatus"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { Question } from 'src/models/Question'
import { QuestionStatusList } from 'src/models/QuestionStatus'

export default {
  name: 'StatusChange',
  props: {
    imgPanelVisibility: {
      type: Boolean,
      default () {
        return false
      }
    },
    statuses: { // possible removal for attach exam
      type: QuestionStatusList,
      default: new QuestionStatusList()
    }
  },
  inject: {
    question: {
      from: 'providedQuestion', // this is optional if using the same key for injection
      default: new Question()
    }
  },
  data () {
    return {
      model: '',
      options: [],
      text: '',
      draftBtnLoading: false,
      saveBtnLoading: false,
      selectedStatus: null,
      order: null,
      selectorRules: [
        v => v !== null || 'پر کردن این فیلد الزامی است.'
      ],
      newStatus: {
        changeState: null,
        commentAdded: null
      }
    }
  },
  methods: {
    sendStatus () {
      this.$emit('update', this.newStatus)
      this.newStatus.changeState = ''
      this.newStatus.commentAdded = null
    }
  },
  computed: {
    getLessonById () {
      return id => this.lessons.list.find(item => item.id === id)
    }
  }
}
</script>

<style scoped lang="scss">
.question-details {
  margin-top: 40px;
  font-style: normal;
  font-weight: 500;
  font-size: 16px;
  line-height: 28px;
  text-align: right #{"/* rtl:ignore */"};
  color: #23263B;

  .default-details-container {
    .detail-box {
      margin-top: 10px;

      .detail-box-title {
        margin-bottom: 5px;
      }
    }
  }
  .detail-box-last-of-row-1 {
    display: flex ;
    //align-items: end ;
    margin-bottom: 2px ;
  }

  .details-container-1 {
    .detail-box {
      padding-right: 12px #{"/* rtl:ignore */"};
      padding-left: 12px #{"/* rtl:ignore */"};
    }

    .detail-box-first {
      padding-right: 0px #{"/* rtl:ignore */"};
    }

    .detail-box-last {
      padding-left: 0px #{"/* rtl:ignore */"};
    }
  }

  .details-container-2 {
    .detail-box {
      padding-right: 12px #{"/* rtl:ignore */"};
      padding-left: 12px #{"/* rtl:ignore */"};

      .input-container {
        .input-box {
          width: 91%;
        }

        .icon-box {
          width: 40px;
          height: 40px;
          background: #FFFFFF;
          border-radius: 10px;
          display: flex;
          align-items: center;
          justify-content: center;
          margin-left: 16px #{"/* rtl:ignore */"};

          .question-details-subject-img {
            height: 24px;
            max-width: 24px;
          }
        }
      }
    }

    .detail-box-first {
      padding-right: 0px #{"/* rtl:ignore */"};
    }

    .detail-box-last {
      padding-right: 0px #{"/* rtl:ignore */"};
      width: 200px;
      //margin-right: 132px #{"/* rtl:ignore */"};
    }

    .detail-box-last-of-row {
      padding-left: 0px #{"/* rtl:ignore */"};
      margin-top: 43px;
      text-align: end;
      display: flex;
      align-items: center;

      .default-detail-btn {
        width: 144px;
        height: 40px;
        border-radius: 10px;
        font-size: 14px;
        line-height: 24px;
        text-align: center;
      }

      .draft-btn {
        background: #FFFFFF;
        margin-left: 16px #{"/* rtl:ignore */"};
        font-weight: normal;
        color: #23263B;
      }

      .save-btn {
        background: #9690E4;
        font-weight: 500;
        color: #FFFFFF;
      }
    }
  }
}
</style>

<style lang="scss">
.question-details {
  .default-details-container {
    .detail-box {
      .q-field__control {
        height: 42px;
      }
      .q-field {
        background: #FFFFFF;
        border-radius: 10px;
        line-height: 24px;
        height: 40px;
        min-height: 40px;
        .q-field__marginal {
          height: 40px;
        }
        .q-field__inner {
          padding-right: 16px;
          padding-left: 16px;
        }
      }
      .q-field--auto-height .q-field__native {
        min-height: 40px;
        color: #65677F;
      }
      .q-field--auto-height .q-field__control, .q-field--auto-height .q-field__native {
        min-height: 40px;
        color: #65677F;
      }
      .q-field__control::before, .q-field__control::after {
        display: none;
      }
      .q-field__native, .q-field__prefix, .q-field__suffix, .q-field__input {
        color: #65677F;
      }
    }
  }
}
.q-menu {
  // I'm in charge of this one and did this on purpose, if you need to change this please let me know.TU
  background: #FFFFFF;
  border-radius: 10px;
}
</style>
