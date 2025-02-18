<template>
  <div>
    <q-card class="custom-card q-pa-none">
      <div class="filter-card-container">
        <div class="filter-header">
          <q-card-section class="header-title-container q-pa-none">
            <div class="header-title">
              فیلترهای اعمال شده
            </div>
          </q-card-section>
          <q-card-actions class="delete-all-container q-pa-none">
            <q-btn flat
                   rounded
                   color="primary"
                   class="delete-all"
                   @click="deleteAllFilters">
              حذف همه
            </q-btn>
          </q-card-actions>
        </div>
        <div>
          <q-card-actions class="filter-container q-pa-none">
            <q-chip
              v-for="(item, index) in selectedFiltersTitle"
              :key="index"
              v-model="selectedFiltersTitle[index]"
              class="filter-items"
              removable
              @remove="deleteFilterObject(item)"
            >
              {{item}}
            </q-chip>
          </q-card-actions>
        </div>
      </div>
    </q-card>
    <div class="filter-options-section">
      <question-filter-expansion
        header-title="درس و مبحث"
      >
        <q-checkbox
          v-model="searchSingleNode"
          class="q-ml-md"
          right-label
          label="جستجوی تک گره"
          @update:model-value="onSearchSingleNode"
        />
        <tree-component
          ref="tree"
          :key="treeKey"
          tick-strategy="strict"
          :get-node-by-id="getNodeById"
          @ticked="tickedData"
          @lazy-loaded="getExpandedTree"
        />
      </question-filter-expansion>
      <!--      header-title="مرجع"-->
      <question-filter-expansion
        header-title="طراح سوال"
      >
        <q-option-group
          v-model="selectedReference"
          type="checkbox"
          :options="filterQuestions.reference_type.map(option => {
            return {
              label: option.value,
              value: option
            }
          })"
          @update:model-value="onChangeReference"
        />
        <div v-if="filterQuestions.reference_type.length === 0"> هیچ مرجعی ایجاد نشده است</div>
      </question-filter-expansion>

      <question-filter-expansion
        header-title="سال انتشار"
      >
        <q-option-group
          v-model="selectedYears"
          type="checkbox"
          :options="filterQuestions.year_type.map(option => {
            return {
              label: option.value,
              value: option
            }
          })"
          @update:model-value="onChangeYears"
        />
        <div v-if="filterQuestions.year_type.length === 0"> هیچ سال انتشاری ایجاد نشده است</div>
      </question-filter-expansion>

      <question-filter-expansion
        v-if="showMajorList"
        header-title="رشته تحصیلی"
      >
        <q-option-group
          v-model="selectedMajors"
          type="checkbox"
          :options="filterQuestions.major_type.map(option => {
            return {
              label: option.value,
              value: option
            }
          })"
          @update:model-value="onChangeMajors"
        />
        <div v-if="filterQuestions.major_type.length === 0"> هیچ رشته تحصیلی ایجاد نشده است</div>

      </question-filter-expansion>

      <question-filter-expansion
        header-title="درجه سختی"
      >
        <q-option-group
          v-model="selectedLevels"
          type="checkbox"
          :options="filterQuestions.levels.map(option => {
            return {
              label: option.value,
              value: option.value
            }
          })"
          @update:model-value="onChangeLevels"
        />
        <div v-if="filterQuestions.levels.length === 0"> هیچ درجه سختی ایجاد نشده است</div>

      </question-filter-expansion>

      <question-filter-expansion
        header-title="وضعیت سوال"
      >
        <q-option-group
          v-model="selectedStatuses"
          type="checkbox"
          :options="filterQuestions.statuses.map(option => {
            return {
              label: option.display_title,
              value: option.display_title
            }
          })"
          @update:model-value="onChangeStatuses"
        />
        <div v-if="filterQuestions.statuses.length === 0"> هیچ درجه سختی ایجاد نشده است</div>

      </question-filter-expansion>

      <question-filter-expansion
        header-title="نوع سوال"
      >
        <q-option-group
          v-model="selectedTypes"
          type="checkbox"
          :options="filterQuestions.types.map(option => {
            return {
              label: option.value,
              value: option.value
            }
          })"
          @update:model-value="onChangeTypes"
        />
        <div v-if="filterQuestions.types.length === 0"> هیچ نوع سوالی ایجاد نشده است</div>

      </question-filter-expansion>
    </div>
  </div>
</template>

<script>
import { mixinTree } from 'src/mixin/Mixins'
import TreeComponent from 'components/Tree/Tree'
import QuestionFilterExpansion from 'components/Question/QuestionBank/QuestionFilterExpansion'

export default {
  name: 'QuestionBankFilter',
  components: { QuestionFilterExpansion, TreeComponent },
  mixins: [mixinTree],
  props: {
    filterQuestions: {
      type: Object,
      default: () => {
        return {
          reference_type: null
        }
      }
    },
    rootNodeIdToLoad: {
      type: String,
      default: () => {
        return ''
      }
    },
    nodeIdsToTick: {
      type: Array,
      default: () => {
        return []
      }
    },
    initialLoadMode: {
      type: Boolean,
      default: () => {
        return true
      }
    },
    showMajorList: {
      type: Boolean,
      default: () => {
        return true
      }
    }
  },
  emits: [
    'onFilter',
    'tagsChanged',
    'deleteFilter'
  ],
  data () {
    return {
      treeKey: 0,
      searchSingleNode: false,
      check: false,
      selectedReference: [],
      selectedYears: [],
      selectedMajors: [],
      selectedLevels: [],
      selectedTypes: [],
      selectedTags: [],
      selectedStatuses: [],
      filtersData: {
        tags: [],
        reference: [],
        majors: [],
        level: [],
        years: [],
        types: [],
        statuses: [],
        tags_with_childrens: 1
      }
    }
  },
  watch: {
    'selectedQuestions.length': {
      handler(newValue, oldValue) {
        this.exam.questions.list = []
        this.exam.questions.list = this.selectedQuestions
        this.questionListKey = Date.now()
      }
    },
    rootNodeIdToLoad: {
      deep: true,
      handler(newVal) {
        this.showTreeModalNode(newVal)
      }
    },
    nodeIdsToTick: {
      deep: true,
      handler(newVal) {
        // console.log('nodeIdsToTick')
        // const tags = newVal.map(function(key) {
        //   return {
        //     id: key
        //   }
        // })
        // this.changeFilterData('tags', tags)
      }
    }
  },
  computed: {
    selectedFiltersTitle () {
      const filtersDataKey = Object.keys(this.filtersData)
      const titles = []
      filtersDataKey.forEach(key => {
        const filterGroup = this.filtersData[key]
        if (typeof filterGroup !== 'number') {
          filterGroup.forEach(filterItem => {
            const title = filterItem.title || filterItem.label || filterItem.value
            titles.push(title)
          })
        }
      })

      return titles
    }
  },
  created () {
    if (this.initialLoadMode) {
      this.showTreeModalNode(this.rootNodeIdToLoad)
    }
  },
  methods: {
    setNodesTicked (nodeIds) {
      this.$refs.tree.setNodesTicked(nodeIds, true)
    },
    getExpandedTree(tree) {
      const currentTreeNodeIds = tree.map(node => node.id)
      const availableIdsToTick = []
      this.nodeIdsToTick.forEach(idToTick => {
        const foundedIds = currentTreeNodeIds.filter(key => key === idToTick)
        availableIdsToTick.push(...foundedIds)
      })
      if (availableIdsToTick.length > 0) {
        this.setNodesTicked(availableIdsToTick)
      }
    },
    showTreeModalNode (NodeId) {
      const nodeToLoadTreeFrom = NodeId ? this.getNode(NodeId) : this.getRootNode('test')
      this.treeKey += 1
      this.showTree('tree', nodeToLoadTreeFrom)
        .then(() => {
        })
    },
    getFilters () {
      return this.filtersData
    },
    onUpdateFilterData () {
      this.$emit('onFilter', this.filtersData)
    },
    changeFilterData (key, value) {
      this.filtersData[key] = value
      this.onUpdateFilterData()
    },
    onChangeReference (value) {
      this.changeFilterData('reference', value)
    },
    onChangeLevels (value) {
      this.changeFilterData('level', value)
    },
    onChangeYears (value) {
      this.changeFilterData('years', value)
    },
    onChangeMajors (value) {
      this.changeFilterData('majors', value)
    },
    onChangeStatuses(value) {
      this.changeFilterData('statuses', value)
    },
    onChangeTypes (value) {
      this.changeFilterData('types', value)
    },
    onSearchSingleNode(value) {
      const sendData = value ? 0 : 1
      this.changeFilterData('tags_with_childrens', sendData)
    },
    tickedData (value) {
      this.$emit('tagsChanged', value)
      this.changeFilterData('tags', value)
      // this.filtersData.tags = value
      // value.forEach(val => {
      //   if (typeof val === 'string') {
      //     this.filtersData.tags.push(val)
      //   } else {
      //     this.filtersData.tags.push(val)
      //   }
      // })
      // this.onUpdateFilterData()
    },
    deleteFilterObject (item) {
      this.$emit('deleteFilter', item)
    },
    deleteAllFilters () {
      this.filtersData.tags.splice(0, this.filtersData.tags.length)
      this.filtersData.reference.splice(0, this.filtersData.reference.length)
      this.filtersData.level.splice(0, this.filtersData.level.length)
      this.filtersData.years.splice(0, this.filtersData.years.length)
      this.filtersData.majors.splice(0, this.filtersData.majors.length)
      this.showTreeModalNode(this.rootNodeIdToLoad)
      // this.QuestionFilters.splice(0, this.QuestionFilters.length)
      this.onUpdateFilterData()
    }
  }
}
</script>
<style lang="scss" scoped>
.custom-card {
  border-radius: 16px;
  @media only screen and (max-width: 1023px) {
    border-radius: 12px !important;
  }
  @media only screen and (max-width: 599px) {
    border-radius: 8px !important;
  }
}
.filter-card-container {
  padding: 20px 23px 16px 24px;
  margin-bottom: 24px;

  .filter-header {
    padding-bottom: 11px;
    display: grid;
    grid-template-columns: auto 76px;
    @media only screen and (max-width: 1439px) {
      grid-template-columns: auto 68px;
    }

    .delete-all-container {
      font-style: normal;
      font-weight: normal;
      font-size: 12px;
      line-height: 21px;
      position: relative;
      top: -5px;
      .delete-all {
        padding: 0;
      }
    }

    .header-title-container {
      .header-title {
        font-style: normal;
        font-weight: 500;
        font-size: 16px;
        line-height: 28px;
        color: #23263B;
      }
    }
  }

  .filter-container {
    .filter-items {
      margin-right: 4px;
      margin-bottom: 4px;
      display: flex;
    }
  }

}

.filter-options-section {

  .filter-option-container {
    width: 500px;
    display: flex;
    //padding: 20px 24px 20px 24px;
    justify-content: space-between;

    .filter-option-title {
      font-style: normal;
      font-weight: 500;
      font-size: 16px;
      line-height: 28px;
      color: #23263B;
    }

    .filter-option-drop-down-btn {
      position: relative;
      top: -3px;
    }
  }
}

@media only screen and (max-width: 1439px) {
  .filter-card-container {
    padding: 20px 16px !important;

    .filter-header {
      .delete-all-container {
        width: 90px;
      }
    }
  }
}

@media only screen and (max-width: 1023px) {
  .filter-card-container {
    padding: 20px 16px !important;
  }
  .filter-options-section {
    display: none;
  }
}

@media only screen and (max-width: 599px) {
  .filter-card-container {
    padding: 12px 16px !important;
  }
}
</style>

<style lang="scss">
.filter-options-section {
  .q-expansion-item__container {
    .q-focus-helper {
      background: none !important;
    }
  }
}
</style>
