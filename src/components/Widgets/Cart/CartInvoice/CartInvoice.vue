<template>
  <div
    class="invoice-container">
    <q-card class="invoice-cart">
      <q-card-section class="invoice-total-price-section invoice-cart-section">
        <div
          class="total-shopping-cart price-section"
        >
          <div class="title">جمع سبد خرید{{ `(${cart.count})` }}</div>
          <div
            v-if="loading"
            class="loading-spinner"
          >
            <q-spinner-tail
              color="orange"
              size="2em"
            />
          </div>
          <div
            v-else
            class="price"
          >
            {{ totalBasePrice }}
            <span class="iran-money-unit">تومان</span>
          </div>
        </div>

        <!--        <div class="wallet-credit price-section">-->
        <!--          <div class="title">استفاده از کیف پول</div>-->
        <!--          <div-->
        <!--            v-if="loading"-->
        <!--            class="loading-spinner"-->
        <!--          >-->
        <!--            <q-spinner-tail-->
        <!--              color="orange"-->
        <!--              size="2em"-->
        <!--            />-->
        <!--          </div>-->
        <!--          <div-->
        <!--            v-else-->
        <!--            class="price"-->
        <!--          >-->
        <!--            {{ amountUsingWallet}}-->
        <!--            <span class="iran-money-unit">تومان</span>-->
        <!--          </div>-->
        <!--        </div>-->

        <div
          v-if="discountInPercent"
          class="purchase-profit price-section"
        >
          <div class="title">سود شما از خرید</div>
          <div
            v-if="loading"
            class="loading-spinner"
          >
            <q-spinner-tail
              color="orange"
              size="2em"
            />
          </div>
          <div
            v-else
            class="price"
          >
            {{ `(${discountInPercent}٪) ` + totalDiscount }}
            <span class="iran-money-unit">تومان</span>
          </div>
        </div>

        <q-separator class="invoice-separator" />
      </q-card-section>

      <!--      <q-card-section class="invoice-coupon-section invoice-cart-section">-->
      <!--        <div class="enter-coupon-code">-->
      <!--          <div class="title">کد تخفیف:</div>-->

      <!--          <q-input-->
      <!--            v-model="couponValue"-->
      <!--            type="text"-->
      <!--            label="کد تخفیف خود را وارد کنید"-->
      <!--            class="coupon-input"-->
      <!--            outlined-->
      <!--          >-->
      <!--            <template v-slot:append>-->
      <!--              <q-btn label="ثبت"-->
      <!--                     flat />-->
      <!--            </template>-->
      <!--          </q-input>-->
      <!--        </div>-->

      <!--        <q-separator class="invoice-separator" />-->
      <!--      </q-card-section>-->

      <q-card-section class="payment-section invoice-cart-section">
        <div class="final-price price-section">
          <div class="title">مبلغ نهایی</div>
          <div
            v-if="loading"
            class="loading-spinner"
          >
            <q-spinner-tail
              color="orange"
              size="2em"
            />
          </div>
          <div
            v-else
            class="price"
          >
            {{ totalFinalPrice }}
            <span class="iran-money-unit">تومان</span>
          </div>
        </div>

        <div
          v-if="isUserLogin"
          class="payment-gateway"
        >
          <p class="payment-title">درگاه پرداخت</p>

          <div
            v-if="loading"
            class="loading-spinner"
          >
            <q-spinner-tail
              color="orange"
              size="3em"
            />
          </div>

          <div
            v-else
            class="banks-gateway-list row"
          >
            <div class="bank-gateway-container col-lg-6  col-md-12 col-sm-6 col-xs-12">
              <div
                class="bank-gateway"
                @click="clickOnGateway"
              >
                <div class="bank-icon-container">
                  <q-img
                    src="https://nodes.alaatv.com/aaa/landing/Banklogos/saman.png"
                    class="bank-icon"
                  />
                </div>
                <q-checkbox
                  v-model="selectedBank"
                  dir="ltr"
                  label="بانک سامان"
                  checked-icon="radio_button_checked"
                  unchecked-icon="radio_button_unchecked"
                  :class="{'checked-check-box': selectedBank}"
                />
              </div>
            </div>
          </div>

          <!--          <div class="payment-description">-->
          <!--            <p class="title">توضیحات</p>-->

          <!--            <q-input-->
          <!--              v-model="shoppingDescription"-->
          <!--              type="text"-->
          <!--              label="اگر توضیحی درباره ی محصول دارید اینجا بنویسید"-->
          <!--              class="payment-description-input"-->
          <!--              outlined-->
          <!--            />-->
          <!--          </div>-->

          <div class="payment-button-container payment-button-container-desktop">
            <div
              class="payment-button payment-button-desktop-view"
              :class="{ 'payment-button-disable': !selectedBank}"
              @click="payment"
            >
              پرداخت و ثبت نهایی
            </div>
          </div>
        </div>

        <q-separator
          v-if="!isUserLogin"
          class="invoice-separator"
        />
      </q-card-section>

      <q-card-section
        v-if="!isUserLogin"
        class="login-section invoice-cart-section"
      >
        <p class="title">برای ادامه ثبت سفارش، به حساب کاربری خود وارد شوید </p>

        <q-input
          v-model="userEnteredLoginInfo.mobile"
          type="text"
          label="شماره موبایل خود را وارد کنید"
          class="login-input"
          outlined
        />

        <q-input
          v-model="userEnteredLoginInfo.password"
          type="password"
          label="رمز عبور خود را وارد کنید"
          class="login-input"
          outlined
        />

        <p class="no-account">
          حساب کاربری ندارید؟
          <router-link to="/"
                       class="sign-in">ثبت نام کنید
          </router-link>
        </p>

        <div
          class="sign-in-button"
          @click="login"
        >
          ورود به حساب کاربری
        </div>
      </q-card-section>
    </q-card>
    <div
      class="payment-button-container"
    >
      <div
        class="final-price price-section"
      >
        <div class="title">مبلغ نهایی:</div>
        <div
          v-if="loading"
          class="loading-spinner"
        >
          <q-spinner-tail
            color="orange"
            size="2em"
          />
        </div>
        <div
          v-else
          class="price"
        >
          {{ totalFinalPrice }}
          <span class="iran-money-unit">تومان</span>
        </div>
      </div>
      <div
        class="payment-button payment-button-mobile-view"
        :class="{ 'payment-button-disable': !selectedBank}"
        @click="payment"
      >
        پرداخت
      </div>
    </div>
    <q-dialog v-model="transition"
              persistent>
      <q-card>
        <q-card-section class="row items-center">
          <span class="q-ml-lg">در حال انتقال به درگاه پرداخت،لطفا منتظر بمانید</span>
        </q-card-section>
        <q-linear-progress
          query
          rounded
          color="positive"
          class="q-mt-sm"
        />
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { Cart } from 'src/models/Cart'

export default {
  name: 'CartInvoice',
  props: {
    data: {
      type: Object,
      default() {
        return {}
      }
    },
    cart: {
      type: Object,
      default: () => new Cart()
    }
  },

  data() {
    return {
      couponValue: null,
      userEnteredLoginInfo: {
        password: '',
        mobile: ''
      },
      selectedBank: true,
      shoppingDescription: '',
      loading: false,
      transition: false
    }
  },

  created() {},

  computed: {
    totalFinalPrice() {
      return this.getPriceFormat('final') ? this.getPriceFormat('final') : 0
    },

    totalBasePrice() {
      return this.getPriceFormat('base')
    },

    totalDiscount() {
      return this.getPriceFormat('discount')
    },

    discountInPercent() {
      return this.cart.price?.discountInPercent()
    },

    isUserLogin() {
      return this.$store.getters['Auth/isUserLogin']
    },

    amountUsingWallet () {
      return this.cart.pay_by_wallet.toLocaleString()
    }
  },

  methods: {
    cartReview() {
      this.$store.commit('loading/loading', true)
      this.$store.dispatch('Cart/reviewCart')
        .then(() => {
          this.$store.dispatch('loading/overlayLoading', false)
        })
    },

    payment() {
      if (!this.selectedBank) {
        return
      }
      this.transition = true
      this.$store.commit('loading/loading', true)
      this.$store.dispatch('Cart/paymentCheckout')
        .then((response) => {
          if (response.data.data.url !== undefined) {
            window.open(response.data.data.url, '_self')
            this.$store.commit('loading/loading', false)
          } else {
            this.$q.notify({
              type: 'negative',
              message: 'در انتقال به درگاه مشکلی پیش آمده (لینک درگاه شناسایی نشد)',
              position: 'top'
            })
            this.$store.commit('loading/loading', false)
          }
        }).catch(() => {
          this.$store.commit('loading/loading', false)
        })
    },

    login() {
      this.$store.dispatch('Auth/login', this.userEnteredLoginInfo)
        .then(() => {
          if (this.isUserLogin) {
            this.cartReview()
          }
        })
    },

    getPriceFormat(priceKey) {
      return this.cart.price.toman(priceKey, null)
    },

    clickOnGateway() {
      this.selectedBank = !this.selectedBank
    }
  }
}
</script>

<style lang="scss" scoped>
.invoice-container {
  margin-left: 30px;

  @media screen and (max-width: 1023px) {
    margin-left: 0;
  }
  @media screen and (max-width: 599px) {
    margin-bottom: 70px;
  }

  .invoice-cart {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30px;
    background: #FFFFFF;
    box-shadow: -2px -4px 10px rgba(255, 255, 255, 0.6), 2px 4px 10px rgba(112, 108, 162, 0.05);
    border-radius: 16px;
    width: 100%;

    @media screen and (max-width: 1439px) {
      padding: 24px;
    }

    @media screen and (max-width: 1023px) {
      padding: 20px;
    }

    &:deep(.q-separator--horizontal) {
      height: 1.3px;
      border: none;
      background: #EFF4FC;
      margin-bottom: 20px;

      @media screen and (max-width: 1439px) {
        margin-bottom: 14px;
      }

      @media screen and (max-width: 1023px) {
        margin-bottom: 16px;
      }

      @media screen and (max-width: 599px) {
        margin-bottom: 12px;
      }
    }

    .invoice-cart-section {
      padding: 0;
      width: 100%;

      .iran-money-unit {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 22px;
        margin-left: 6px;
      }

      &.invoice-total-price-section {
        .price-section {
          display: flex;
          justify-content: space-between;
          font-style: normal;
          font-weight: 400;
          font-size: 16px;
          line-height: 25px;
          color: #23263B;
          margin-bottom: 20px;

          @media screen and (max-width: 1439px) {
            margin-bottom: 14px;
          }

          @media screen and (max-width: 1023px) {
            margin-bottom: 16px;
          }

          &.purchase-profit {
            color: #E86562;
          }

          .title {
            letter-spacing: -0.03em;

            @media screen and (max-width: 599px) {
              font-size: 14px;
            }
          }

          .price {
            letter-spacing: -0.05em;
          }
        }
      }

      &.invoice-coupon-section {
        .enter-coupon-code {
          display: flex;
          align-items: center;
          margin-bottom: 20px;
          justify-content: space-between;

          @media screen and (max-width: 1439px) {
            margin-bottom: 14px;
          }

          @media screen and (max-width: 1023px) {
            margin-bottom: 16px;
          }

          @media screen and (max-width: 599px) {
            margin-bottom: 12px;
          }

          .title {
            font-style: normal;
            font-weight: 400;
            font-size: 14px;
            line-height: 25px;
            letter-spacing: -0.03em;
            color: #23263B;
            margin-right: 16px;
            min-width: 72px;

            @media screen and (max-width: 1439px) {
              margin-right: 4px;
            }

            @media screen and (max-width: 1023px) {
              margin-right: 36px;
            }

            @media screen and (max-width: 599px) {
              font-size: 14px;
              margin-right: 14px;
            }
          }

          .coupon-input {
            @media screen and (max-width: 1023px) {
              width: 100%;
            }

            &:deep(.q-field__control) {
              height: 40px;
              border: 1.3px solid #E7ECF4;
              border-radius: 8px;
              padding: 0 16px;
              width: 286px;

              @media screen and (max-width: 1439px) {
                padding: 0 12px;
                //min-width: 174px;
                width: 100%;
              }

              @media screen and (max-width: 1023px) {
                padding: 0 16px;
                //min-width: 392px;
              }

              @media screen and (max-width: 599px) {
                padding: 0 12px;
                //min-width: 196px;
              }
            }

            &:deep(.q-field__append) {
              height: 40px;
              width: 45px;
            }

            &:deep(.q-field__label) {
              font-style: normal;
              font-weight: 400;
              font-size: 12px;
              line-height: 19px;
              letter-spacing: -0.05em;
              color: #9092A7;
              margin: -8px 0;
            }

            &:deep(.q-btn .q-btn__content) {
              font-style: normal;
              font-weight: 400;
              font-size: 16px;
              line-height: 22px;
              color: #23263B;

              @media screen and (max-width: 1439px) {
                font-size: 14px;
              }
            }

            &:deep(.q-field__inner .q-field__control:before) {
              border: none;
            }

          }
        }
      }

      &.payment-section {
        .final-price {
          color: #4CAF50;
          display: flex;
          justify-content: space-between;
          margin-bottom: 24px;
          align-items: center;

          @media screen and (max-width: 1439px) {
            margin-bottom: 18px;
          }

          @media screen and (max-width: 1023px) {
            margin-bottom: 20px;
          }

          @media screen and (max-width: 599px) {
            margin-bottom: 16px;
          }

          .title {
            font-weight: 600;
            font-size: 18px;
            line-height: 28px;
            letter-spacing: -0.03em;

            @media screen and (max-width: 599px) {
              font-size: 16px;
            }
          }

          .price {
            font-weight: 600;
            font-size: 24px;
            line-height: 37px;
            letter-spacing: -0.05em;

            @media screen and (max-width: 599px) {
              font-size: 20px;
            }
          }
        }

        .payment-gateway {

          .payment-title {
            font-weight: 400;
            font-size: 16px;
            line-height: 25px;
            letter-spacing: -0.03em;
            color: #23263B;
            margin-bottom: 8px;

            @media screen and (max-width: 1439px) {
              margin-bottom: 6px;
            }

            @media screen and (max-width: 1023px) {
              margin-bottom: 8px;
            }

            @media screen and (max-width: 599px) {
              margin-bottom: 6px;
            }
          }

          .loading-spinner {
            display: flex;
            justify-content: center
          }

          .banks-gateway-list {
            margin-bottom: 20px;
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;

            @media screen and (max-width: 1439px) {
              margin-bottom: 8px;
            }

            @media screen and (max-width: 1023px) {
              margin-bottom: 16px;
            }

            @media screen and (max-width: 599px) {
              margin-bottom: 2px;
            }

            .bank-gateway-container {
              @media screen and (max-width: 1023px) {
                padding: 0 6px;
              }

              @media screen and (max-width: 599px) {
                padding: 0;
              }

              .bank-gateway {
                display: flex;
                justify-content: space-between;
                align-items: center;
                box-sizing: border-box;
                height: 74px;
                width: 177px;
                border: 1.3px solid #E7ECF4;
                border-radius: 8px;
                padding: 8px;
                cursor: pointer;

                .checked-check-box {
                  &:deep(.q-icon) {
                    color: #FFB74D;

                  }
                }

                &:deep(.q-checkbox__inner  ) {
                  width: 20px;
                }
                &:deep(.q-checkbox__icon-container ) {
                  width: 20px;
                }

                &:deep(.q-checkbox__label) {
                  font-style: normal;
                  font-weight: 400;
                  font-size: 11px;
                  line-height: 19px;
                  letter-spacing: -0.05em;
                  color: #23263B;
                }

                @media screen and (max-width: 1439px) {
                  width: 100%;
                  margin-bottom: 8px;
                }

                @media screen and (max-width: 1023px) {
                  margin-bottom: 0;
                }

                @media screen and (max-width: 599px) {
                  margin-bottom: 10px;
                }

                .bank-icon-container {
                  min-width: 58px;
                  height: 58px;
                  background: #F4F3FF;
                  border-radius: 5px;
                  margin-right: 8px;
                }

                .select-bank-radio-button {
                  width: 100%;
                  justify-content: space-between;

                  &:deep(.q-radio__bg) {
                    color: #E7ECF4;
                  }

                  &:deep(.q-radio__inner--truthy .q-radio__check) {
                    color: #FFB74D;
                  }

                }
              }
            }
          }

          .payment-description {
            .title {
              font-style: normal;
              font-weight: 400;
              font-size: 16px;
              line-height: 25px;
              letter-spacing: -0.03em;
              color: #23263B;
              margin-bottom: 8px;

              @media screen and (max-width: 1439px) {
                margin-bottom: 6px;
              }

              @media screen and (max-width: 599px) {
                font-size: 14px;
              }
            }

            .payment-description-input {
              margin-bottom: 24px;

              &:deep(.q-field__control) {
                height: 65px;
                border: 1.3px solid #E7ECF4;
                border-radius: 8px;
              }

              &:deep(.q-field__label ) {
                font-style: normal;
                font-weight: 400;
                font-size: 12px;
                line-height: 19px;
                letter-spacing: -0.05em;
                color: #9092A7;
                margin: -8px 0;
              }

              &:deep(.q-field__label .q-field__native) {
                padding: 12px 16px;
              }

              &:deep(.q-field__inner .q-field__control:before) {
                border: none;
              }
            }
          }
        }
      }

      &.login-section {
        .title {
          font-weight: 400;
          font-size: 16px;
          line-height: 25px;
          letter-spacing: -0.003em;
          color: #23263B;
          margin-bottom: 14px;
        }

        .login-input {
          margin-bottom: 14px;

          &:deep(.q-field__control) {
            width: 374px;
            height: 40px;
            background: #F6F9FF;
            border-radius: 8px;
            padding: 0 16px;
          }

          &:deep(.q-field__label) {
            font-style: normal;
            font-weight: 400;
            font-size: 12px;
            line-height: 19px;
            text-align: right;
            letter-spacing: -0.003em;
            color: #ADAFC1;
            margin: -8px 0;
          }

          &:deep(.q-field__inner .q-field__control:before) {
            border: none;
          }
        }

        .no-account {
          text-align: right;
          font-style: normal;
          font-weight: 600;
          font-size: 12px;
          line-height: 19px;
          letter-spacing: -0.003em;
          color: #23263B;

          .sign-in {
            color: #8075DC;
            margin-bottom: 20px;
            text-decoration: none;
            cursor: pointer;
          }
        }

        .sign-in-button {
          display: flex;
          justify-content: center;
          align-items: center;
          height: 56px;
          background: #9690E4;
          border-radius: 8px;
          font-style: normal;
          font-weight: 600;
          font-size: 16px;
          line-height: 25px;
          letter-spacing: -0.03em;
          color: #FFFFFF;
          cursor: pointer;
        }
      }
    }

  }

  .payment-button-container {
    &.payment-button-container-desktop {
      display: flex;
      @media screen and (max-width: 599px) {
        display: none;
      }
    }

    @media screen and (max-width: 599px) {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      display: flex;
      justify-content: space-between;
      padding: 13px 19px;
      background: #FFFFFF;
      box-shadow: 0px -6px 10px rgba(112, 108, 161, 0.07);
      border-radius: 16px 16px 0 0;
    }

    .final-price {
      display: none;
      color: #434765;
      @media screen and (max-width: 599px) {
        display: flex;
      }

      .title {
        font-style: normal;
        font-weight: 400;
        font-size: 14px;
        line-height: 22px;
        margin-right: 4px;
      }

      .price {
        font-style: normal;
        font-weight: 600;
        font-size: 16px;
        line-height: 25px;
        letter-spacing: -0.05em;
      }

      .iran-money-unit {
        font-style: normal;
        font-weight: 400;
        font-size: 12px;
        line-height: 19px;
      }
    }

    .payment-button {
      justify-content: center;
      align-items: center;
      height: 56px;
      background: #4CAF50;
      border-radius: 8px;
      font-style: normal;
      font-weight: 600;
      font-size: 16px;
      line-height: 25px;
      letter-spacing: -0.03em;
      color: #FFFFFF;
      cursor: pointer;
      width: 100%;

      &.payment-button-disable {
        opacity: 0.5;
        cursor: default;
      }

      @media screen and (max-width: 599px) {
        width: 104px;
        height: 36px;
        font-style: normal;
        font-weight: 600;
        font-size: 14px;
        line-height: 22px;
      }

      &.payment-button-mobile-view {
        display: none;
        @media screen and (max-width: 599px) {
          display: flex;
        }
      }

      &.payment-button-desktop-view {
        display: flex;
        @media screen and (max-width: 599px) {
          display: none;
        }
      }
    }
  }

}
</style>
