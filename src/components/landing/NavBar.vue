<template>
  <div class="nav-bar-position">
    <div class="nav-bar flex items-center justify-between landing-width">
      <div class="logo">
        <svg width="19"
             height="38"
             viewBox="0 0 19 38"
             fill="none"
             xmlns="http://www.w3.org/2000/svg">
          <path
            d="M0.742667 16.4967C0.625835 16.5765 0.530457 16.6826 0.464592 16.8059C0.398728 16.9292 0.364314 17.0662 0.364258 17.2051L0.364258 31.7653C0.364258 31.7653 0.364258 37.5051 5.56987 38.0003H10.9927C11.0556 38.0005 11.117 37.9812 11.1679 37.9451C11.2188 37.909 11.2566 37.858 11.2757 37.7996C11.2948 37.7411 11.2942 37.6783 11.2741 37.6202C11.254 37.5621 11.2154 37.5117 11.1638 37.4765C9.92338 36.6291 7.84078 34.746 8.08236 31.7653V12.0322C8.08238 11.9794 8.0675 11.9276 8.03936 11.8824C8.01122 11.8373 7.97089 11.8005 7.92277 11.7762C7.87465 11.7518 7.82059 11.7408 7.76648 11.7443C7.71237 11.7478 7.66028 11.7657 7.6159 11.7961L0.742667 16.4967Z"
            fill="#FFC107" />
          <path
            d="M18.4268 8.20392C18.427 8.291 18.3994 8.37599 18.3479 8.44714C18.2963 8.51829 18.2234 8.57208 18.139 8.60106C18.0547 8.63004 17.9632 8.63278 17.8773 8.6089C17.7913 8.58503 17.715 8.53571 17.6591 8.46778C16.1423 6.63177 14.2272 6.59922 12.8613 6.90926C12.0484 7.09396 11.2885 7.45413 10.6057 7.92072L5.58613 11.3575L2.4866 13.4798L0.830722 14.6113C0.786341 14.6416 0.734255 14.6595 0.680143 14.663C0.626031 14.6666 0.571969 14.6555 0.523851 14.6312C0.475733 14.6068 0.435405 14.57 0.407263 14.5249C0.379121 14.4798 0.364246 14.4279 0.364258 14.3751V7.91588C0.364007 7.45154 0.478814 6.99401 0.698932 6.58212C0.919049 6.17023 1.23798 5.81615 1.62863 5.54995L2.4866 4.96682L5.58839 2.85595L7.90535 1.28204C8.73352 0.718701 9.66645 0.300046 10.6599 0.115345C11.9432 -0.124326 13.6402 -0.0737531 15.2785 1.08502C17.9955 3.00811 18.4177 6.36439 18.4268 8.20392Z"
            fill="#FFC107" />
          <path
            d="M13.6448 16.8265C15.895 16.8265 17.7192 15.05 17.7192 12.8585C17.7192 10.667 15.895 8.8905 13.6448 8.8905C11.3945 8.8905 9.57031 10.667 9.57031 12.8585C9.57031 15.05 11.3945 16.8265 13.6448 16.8265Z"
            fill="#FFC107" />
          <path
            d="M13.6448 26.3856C15.895 26.3856 17.7192 24.6091 17.7192 22.4176C17.7192 20.2262 15.895 18.4496 13.6448 18.4496C11.3945 18.4496 9.57031 20.2262 9.57031 22.4176C9.57031 24.6091 11.3945 26.3856 13.6448 26.3856Z"
            fill="#FFC107" />
          <path
            d="M13.6448 36.0165C15.895 36.0165 17.7192 34.24 17.7192 32.0485C17.7192 29.857 15.895 28.0805 13.6448 28.0805C11.3945 28.0805 9.57031 29.857 9.57031 32.0485C9.57031 34.24 11.3945 36.0165 13.6448 36.0165Z"
            fill="#FFC107" />
        </svg>
      </div>
      <div class="nav-item-box flex xs-hide">
        <div class="nav-item"
             @click="onClickRegisterBtn"
        >
          آزمون سه‌آ
        </div>
        <div class="nav-item middle"
             @click="onClickConditionsBtn"
        >
          شرایط برگزاری آزمون
        </div>
        <div class="nav-item"
             @click="onClickFaqsBtn"
        >
          سوالات متداول
        </div>
      </div>
      <div>
        <q-btn
          unelevated
          class="submit-btn-style sign-up-btn"
          label="ثبت نام آزمون"
          @click="onClickRegisterBtn"
        />
        <q-btn
          unelevated
          class="submit-btn-style"
          label="آزمون های من"
          @click="onClickUserExamListBtn"
        />
      </div>
    </div>
  </div>

</template>

<script>
export default {
  name: 'NavBar',
  emits: ['onClickUserExamListBtn', 'onClickRegisterBtn', 'onClickConditionsBtn', 'onClickFaqsBtn'],
  methods: {
    onClickUserExamListBtn () {
      this.$emit('onClickUserExamListBtn')
    },
    onClickRegisterBtn () {
      this.$emit('onClickRegisterBtn')
    },
    onClickConditionsBtn () {
      this.$emit('onClickConditionsBtn')
    },
    onClickFaqsBtn () {
      this.$emit('onClickFaqsBtn')
    }
  }
}
</script>

<style lang="scss" scoped>
.nav-bar-position{
  //position: fixed;
  //width: 100%;
}
.nav-bar {
  padding: 20px 0;
  .nav-item-box {
    .nav-item {
      font-style: normal;
      font-weight: 500;
      font-size: 20px;
      line-height: 24px;
      color: #474747;
      cursor: pointer;
      &.middle{
        margin: 0 46px;
      }
    }
  }
  &:deep(.q-btn) {
    .q-btn__content {
      margin: 0;
    }
  }
  .logo{
    //width: 105px;
    width: 200px;
  }
  .submit-btn-style {
    box-sizing: border-box;
    width: 105px;
    height: 40px;
    background: #FFF0C1;
    border: 2px solid #FEA800;
    border-radius: 24px;
    &.sign-up-btn {
      margin-right: 10px;
    }
  }
}
@media screen and (max-width: 1439px){
  .nav-bar {
    padding: 12px 0;
    .nav-item-box {
      .nav-item {
        font-weight: 700;
        font-size: 14px;
        line-height: 24px;
        &.middle{
          margin: 0 25px;
        }
      }
    }
  }
}
@media  screen and (max-width: 1023px) {
  .nav-bar {
    padding: 18px 0;
    .nav-item-box {
      .nav-item {
        &.middle{
          margin: 0 16px;
        }
      }
    }
    .logo{
      width: 50px !important;
    }
  }
}
@media  screen and (max-width: 765px) {
  .nav-bar {
    padding: 23px 0;
    .nav-item-box {
      .nav-item {
        font-weight: 700;
        font-size: 10px;
        line-height: 24px;
        &.middle{
          margin: 0 24px;
        }
      }
    }
    .logo{
      width: 96px;
    }
    .submit-btn-style {
      width: 96px;
      height: 32px;
      font-weight: 700;
      font-size: 14px;
      line-height: 24px;
    }
  }
}
@media  screen and (max-width: 600px) {
  .nav-bar {
    padding: 16px 15px !important;
  }
}
</style>
