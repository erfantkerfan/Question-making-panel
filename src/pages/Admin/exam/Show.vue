<template>
  <div>
    <entity-show
      v-model:value="inputs"
      title="اطلاعات آزمون"
      :api="api"
      :entity-id-key="entityIdKey"
      :entity-param-key="entityParamKey"
      :edit-route-name="editRouteName"
      :index-route-name="indexRouteName"
      :copy-on-click="true"
      @onCopyToClipboard="onCopyToClipboard"
    >
      <template #after-form-builder>
        <div
          v-for="(category , index) in inputs[examCategoriesIndex].value"
          :key="index"
          class="row"
        >
          <q-select
            v-model="category.title"
            class="q-pa-md col-md-4"
            :value="category.id"
            label="دفترچه"
            :options="inputs[examCategoriesIndex].value"
            option-value="title"
            option-label="title"
            emit-value
            map-options
            disable
          />
          <q-input
            v-model="category.order"
            class="q-pa-md col-md-3"
            label="ترتیب"
            disable
          />
          <q-input
            v-model="category.time"
            class="q-pa-md col-md-3"
            label="زمان"
            disable
          />
        </div>
      </template>
    </entity-show>
  </div>
</template>

<script>
import { EntityShow } from 'quasar-crud'
import API_ADDRESS from 'src/api/Addresses'

export default {
  name: 'Show',
  components: {
    EntityShow
  },
  data () {
    return {
      api: API_ADDRESS.exam.base(),
      entityIdKey: 'data.id',
      entityParamKey: 'data.id',
      editRouteName: 'Admin.Exam.Edit',
      indexRouteName: 'Admin.Exam.Index',
      inputs: [
        {
          type: 'formBuilder',
          col: 'col-md-4',
          value: [
            { type: 'file', name: 'photo', responseKey: 'data.photo', label: 'پیش نمایش تصویر', col: 'col-md-6', placeholder: ' ' }
          ]
        },
        {
          type: 'formBuilder',
          col: 'col-md-8',
          value: [
            { type: 'input', name: 'title', responseKey: 'data.title', label: 'عنوان', col: 'col-md-12', placeholder: ' ', filled: true, readonly: true },
            { type: 'dateTime', name: 'start_at', responseKey: 'data.start_at', label: 'زمان شروع', calendarIcon: ' ', col: 'col-md-6', placeholder: 'زمان شروع' },
            { type: 'dateTime', name: 'finish_at', responseKey: 'data.finish_at', label: 'زمان پایان', calendarIcon: ' ', col: 'col-md-6', placeholder: 'زمان پایان' }
          ]
        },
        {
          type: 'Select',
          name: 'type_id',
          responseKey: 'data.type.value',
          label: ' انتخاب نوع آزمون',
          col: 'col-md-3',
          options: [],
          optionValue: 'id',
          optionLabel: 'value',
          placeholder: ' ',
          filled: true
        },
        { type: 'input', name: 'delay_time', responseKey: 'data.delay_time', label: 'زمان تاخیر(دقیقه)', col: 'col-md-3', placeholder: ' ', filled: true },
        { type: 'Checkbox', name: 'enable', responseKey: 'data.enable', label: 'فعال', col: 'col-md-3' },
        { type: 'Checkbox', name: 'is_free', responseKey: 'data.is_free', label: 'رایگان', col: 'col-md-3' },
        { type: 'Checkbox', name: 'is_register_open', responseKey: 'data.is_register_open', label: 'ثبت نام باز است.', col: 'col-md-3' },
        { type: 'Checkbox', name: 'is_open', responseKey: 'data.is_open', label: 'شرکت در آزمون باز است.', col: 'col-md-3' },
        { type: 'Checkbox', name: 'confirm', responseKey: 'data.confirm', label: 'تولید خودکار کارنامه', col: 'col-md-3' },
        { type: 'Checkbox', name: 'generate_questions_automatically', responseKey: 'data.generate_questions_automatically', label: 'تولید خودکار سوال', col: 'col-md-3' },
        { type: 'hidden', name: 'categories', responseKey: 'data.categories', value: [] }
      ]
    }
  },
  methods: {
    onCopyToClipboard (data) {
      this.$q.notify({
        message: 'مقدار ' + data.input.label + ' کپی شد.'
      })
    },
    generateJsonFile (id, withAnswer) {
      this.$store.dispatch('loading/linearLoading', true)
      this.$axios.post(API_ADDRESS.exam.generateExamFile(id, withAnswer))
        .then(() => {
          this.$q.notify({
            type: 'positive',
            message: 'ساخت فایل ' + this.inputs[0].value + ' با موفقیت انجام شد',
            position: 'top'
          })
          this.$store.dispatch('loading/linearLoading', false)
        })
        .catch(() => {
          this.$store.dispatch('loading/linearLoading', false)
        })
    },
    editCoefficient (id) {
      this.$router.push({
        name: 'Admin.Exam.Coefficient.Edit',
        params: {
          id
        }
      })
    },
    getRemoveMessage (row) {
      const title = row.title
      return 'آیا از حذف ' + title + ' اطمینان دارید؟'
    }
  },
  created () {
    this.api += '/' + this.$route.params.id
  },
  computed: {
    examCategoriesIndex () {
      return this.inputs.findIndex(item => item.name === 'categories')
    }
  }
}
</script>

<style scoped>

</style>
