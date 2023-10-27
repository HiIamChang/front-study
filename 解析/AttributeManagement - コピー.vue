<template>
  <div class="service-template">
    <div class="content-top">
      <div class="top-left">
        <div class="top-remove">
          <flux-button
            :label="$t('attributeManagement.button.removeAttribute')"
            :tooltip="$t('attributeManagement.button.removeAttribute')"
            :disabled="isSelect"
            @click="showDelAttribute" />
        </div>
        <div class="top-search">
          <flux-search-field
            width="238"
            :placeholder="$t('attributeManagement.search.attrName')"
            :tooltip="$t('attributeManagement.search.attrName')"
            :value="searchString"
            @ui-search-start-event="search()"
            @ui-search-stop-event="stopSearch()"
            @input="v => searchString = v.detail[0]" />
        </div>
      </div>
      <div class="top-right">
        <flux-button
          type="normal"
          :label="$t('attributeManagement.button.addNewAttribute')"
          :disabled="addButton"
          no-outline="false"
          :tooltip="$t('attributeManagement.button.addNewAttribute')"
          loading="false"
          @click="showAttribute({attrId:'',attrName: ''})" />
      </div>
    </div>
    <div class="content-table">
      <div class="ui-table-main">
        <div class="ui-table-content">
          <flux-table
            :loading="listLoading.toString()"
            :selectable="selectable"
            selection-mode="single"
            width="full"
            :max-height="'720'"
            :required="false"
            :default-sort-options="JSON.stringify({ key: 'name', order: 'asc' })"
            :fields="JSON.stringify(fields)"
            :items="JSON.stringify(newItems)"
            @ui-valid-event="v => v4 = v.detail[0]"
            @flux-table-selected-event="v => selectedList = v.detail[0]">
            <template v-for="field in fields">
              <span
                v-for="(item, row) in newItems"
                :key="field.key + '-' + row"
                :slot="field.key + '-' + row"
                :title="field.key === 'name'? item[field.key] : field.key === 'type'? getLabelByValue(item[field.key]): null "
                class="text-overflow">
                <template v-if="item[field.key] !== null && field.key === 'name'">
                  {{ item[field.key] }}
                </template>
                <template v-else-if="field.key === 'type'">
                  {{ getLabelByValue(item[field.key]) }}
                </template>
                <template v-else-if="field.key === 'tool' && item.id !==0">
                  <div class="menu-class" style="margin: 1.5px 0 1.5px;">
                    <flux-button-menu
                      :items="
                        JSON.stringify([
                          {
                            label: $t('attributeManagement.editTitle'),
                            tooltip: $t('attributeManagement.editTitle'),
                            disabled: false,
                            value: {
                              attrId: item['id'],
                              attrName: item['name'],
                              attrType: item['type'],
                              isDelButton:false
                            },
                          },
                          {
                            label: $t('attributeManagement.button.removeAttribute'),
                            tooltip: $t('attributeManagement.button.removeAttribute'),
                            disabled: false,
                            value: {
                              attrId: item['id'],
                              attrName: item['name'],
                              attrType: item['type'],
                              isDelButton:true
                            },
                          }
                        ])
                      "
                      icon-id="ellipsisvert"
                      disabled="false"
                      @flux-menu-selected-event="v => selectMenu(v.detail[0])" />
                  </div>
                </template>
              </span>
            </template>
          </flux-table>
        </div>
      </div>
    </div>

    <div>
      <span style="font-size: 14px">{{ $t('attributeManagement.attributeCount', [newTotal]) }}</span>
    </div>

    <div v-if="setDialogVisible">
      <flux-modal-dialog
        class="flux-class"
        icon-type="primary"
        width="680"
        height="300"
        :label="dialogTitle"
        movable="false"
        modeless="false">
        <span slot="body">
          <div>
            <div class="item-container">

              <div class="line-item">
                <div class="item-label">
                  <flux-label :label="$t('attributeManagement.search.attrName')" />
                </div>
                <div class="item-input">
                  <flux-text-field
                    ref="attrName"
                    :lang="$i18n.locale"
                    width="648"
                    type="text"
                    required="true"
                    max-length="50"
                    :value="attrName"
                    :custom-validation-message="customValidationMessage"
                    @input="changeInput" />
                </div>
              </div>

              <div class="line-item">
                <div class="item-label">
                  <flux-label :label="$t('attributeManagement.search.type')" />
                </div>
                <div v-if="attrId !==''" class="item-input">
                  <flux-dropdown
                    disabled="true"
                    lang="$i18n.locale"
                    :items="JSON.stringify(typeList)"
                    :value="selectType"
                    width="200"
                    @input="v => selectType = v.detail[0]" />
                </div>
                <div v-else class="item-input">
                  <flux-dropdown
                    lang="$i18n.locale"
                    :items="JSON.stringify(typeList)"
                    :value="selectType"
                    width="200"
                    @input="v => selectType = v.detail[0]" />
                </div>
              </div>

            </div>
          </div>
        </span>
        <span slot="footer">
          <div>
            <div class="footer-buttons" style="margin-left: 520px">
              <flux-button :label="$t('confirmDialog.cancel')" @click="closeSimpleDialog"/>
              <flux-button style="position: absolute;padding-left: 10px;" type="primary" :label="attrId==='' ? $t('attributeManagement.button.addAttribute') : 'OK' "  :disabled="itemTotal > 199 && attrId ==='' ? true : false" @click="attrId ==='' ? saveAttribute() : updateAttribute()"></flux-button>
            </div>
          </div>
        </span>
      </flux-modal-dialog>
    </div>

    <div v-if="setDelDialogVisible">
      <flux-modal-dialog
        icon-id="warning"
        icon-type="alert"
        width="650"
        height="100"
        :label="$t('attributeManagement.delTitle')"
        movable="false"
        modeless="false">
        <span slot="body">
          <div>
            <flux-label :label="delInfo != null ? $t('attributeManagement.delDialogTitle', [delInfo.attrName]) : selectedList.length === 1 ? $t('attributeManagement.delDialogTitle', [selectedList[0].name]) : $t('attributeManagement.delDialogTitle2')" />
          </div>
          <div style="padding-top: 16px;">
            <flux-label :label="$t('attributeManagement.delDialogContent')" />
          </div>
        </span>
        <span slot="footer">
          <div>
            <div class="footer-buttons" style="margin-left: 480px;">
              <flux-button :label="$t('confirmDialog.cancel')" @click="closeDelAttribute"/>
              <flux-button style="position: absolute;padding-left: 10px;" :label="$t('confirmDialog.ok')" type="primary" @click="delAttributeItem"></flux-button>
            </div>
          </div>
        </span>
      </flux-modal-dialog>
    </div>
  </div>
</template>

<script lang="ts">
import utils from '../modules/utils'
import { TranslateResult } from 'vue-i18n'
import { mapActions } from 'vuex'

export enum AttributeValueType {
  string = 'STRING',
  boolean = 'BOOLEAN',
  date = 'DATE',
  float = 'FLOAT',
  integer = 'INTEGER',
  user_group = 'USER_GROUP'
}

export default {
  name: 'AttributeManagement',
  components: {
  },

  props: {
    repositoryId: {
      type: String,
      default: 'ST17447F1DF7F',
    },
    loginUserId: {
      type: String,
      default: '',
    }
  },

  // eslint-disable-next-line @typescript-eslint/ban-types
  data(): Object {
    return {
      listLoading: true,
      setDialogVisible: false,
      setDelDialogVisible: false,
      message: '',
      fields: [
        { key: 'selector', width: '1'},
        { key: 'name', sortable: false, label: this.$t('attributeManagement.search.attrName'), tooltip: this.$t('attributeManagement.search.attrName'), 'width-percentage': '30', 'min-width': '30'},
        { key: 'type', sortable: false, label: this.$t('attributeManagement.search.type'), tooltip: this.$t('attributeManagement.search.type'), 'width-percentage': '30', 'min-width': '30'},
        { key: 'tool', sortable: false, label: '', 'width-percentage': '5', 'min-width': '30' },
      ],
      newItems: [],
      defaultSortOptions: { key: 'name', order: 'asc' },
      selectedList: [],
      v4: '',
      attrName: '',
      customValidationMessage: '',
      typeList: [
        { label: this.$t('attributeManagement.string'), value: AttributeValueType.string },
        { label: this.$t('attributeManagement.boolean'), value: AttributeValueType.boolean},
        { label: this.$t('attributeManagement.date'), value: AttributeValueType.date},
        { label: this.$t('attributeManagement.float'), value: AttributeValueType.float},
        { label: this.$t('attributeManagement.integer'), value: AttributeValueType.integer},
        { label: this.$t('attributeManagement.user_group'), value: AttributeValueType.user_group},
      ],
      selectType: '',
      isSelect: 'false',
      dialogTitle: this.$t('attributeManagement.title'),
      newTotal: 0,
      selectable: 'true',
      searchString: '',
      attrId: '',
      attrLabel: '',
      sortOption: {'key': '', 'order': ''},
      addButton: 'true',
      itemTotal: 0,
      delInfo: null
    }
  },

  watch: {
    selectedList: {
      handler(visible: string): void {
        if (visible.length > 0){
          this.isSelect = 'false'
        } else {
          this.isSelect = 'true'
        }
      },
      immediate: true
    },
  },

  mounted: function (): void {
    this.listLoading = true
    this.getAttributeList({repositoryId: this.repositoryId}).then((e) => {
      this.newItems = e.items
      this.listLoading = false
      this.newTotal = this.newItems.length
      this.itemTotal = e.items.length
      if (this.itemTotal > 199){
        this.addButton = 'true'
      } else {
        this.addButton = 'false'
      }
      this.getSelectableStatus()
    }).catch(() => {
      this.listLoading = false
      this.getSelectableStatus()
    })
  },

  methods: {
    ...mapActions({
      getAttributeList: 'attribute/getAttributeList',
      addAttribute: 'attribute/addAttribute',
      delAttribute: 'attribute/delAttribute',
      filterItems: 'attribute/filterItems',
      editAttribute: 'attribute/editAttribute',
    }),

    inputNameFocus: function (): void {
      if (this.$refs.attrName.vueComponent){
        this.$refs.attrName.focus()
      }
    },
    // xinguizuocheng
    showAttribute: function (e): void {
      if (e.attrId){
        this.dialogTitle = this.$t('attributeManagement.editTitle')
        this.$set(this, 'attrLabel', this.getLabelByValue(e.attrType))
        this.attrName = e.attrName
        this.selectType = e.attrType
        this.attrId = e.attrId
      } else {
        this.dialogTitle = this.$t('attributeManagement.title')
        this.attrName = ''
        this.selectType =''
        this.attrId = ''
      }
      this.$nextTick().then(this.inputNameFocus.bind(this))
      this.setDialogVisible = true

    },
    //deleteddialog
    selectMenu: function (e): void {
      if (e.isDelButton == true){
        this.delInfo = e
        this.setDelDialogVisible = true
      } else {
        this.showAttribute(e)
      }
    },
    // wuyong
    showDelAttribute: function (): void {
      this.setDelDialogVisible = true
    },
    // wuyong
    closeSimpleDialog: function (): void {
      this.setDialogVisible = false
      this.attrName = ''
      this.message = ''
      this.customValidationMessage=''
    },
    // wuyong
    closeDelAttribute: function (): void {
      this.setDelDialogVisible = false
      this.delInfo = null
    },
    // wuyong
    delAttributeItem: function (): void {
      this.setDelDialogVisible = false
      this.listLoading = true
      this.delAttribute({
        repositoryId: this.repositoryId,
        cstmAttiKeyIds: this.delInfo == null ? this.selectedList.map(e => e.id) : [this.delInfo.attrId]
      }).then(() => {
        this.getItems()
        this.delInfo = null
      }).catch(() => {
        this.listLoading = false
      })
    },

    search: function (): void {
      this.listLoading = true
      this.getAttributeList({repositoryId: this.repositoryId, filter: encodeURIComponent(this.searchString)}).then((e) => {
        this.listLoading = false
        this.newItems = e.items
        this.newTotal = e.items.length
        this.getTotalStatus()
        this.getSelectableStatus()
      }).catch(() => {
        this.listLoading = false
      })

    },

    // add wuyong
    saveAttribute: function(): void{
      if (this.$refs.attrName.vueComponent && !this.$refs.attrName.vueComponent.validate()) {
        return
      }
      if (this.customValidationMessage !== '') {
        return
      }
      this.listLoading = true
      let param = {
        repositoryId: this.repositoryId,
        name: this.attrName,
        type: this.selectType
      }
      this.setDialogVisible = false
      this.addAttribute(param).then(() => {
        this.getItems()
      }).catch(() => {
        this.listLoading = false
      })
      this.closeSimpleDialog()
    },

    // wuyong
    updateAttribute: function(): void{
      if (this.$refs.attrName.vueComponent && !this.$refs.attrName.vueComponent.validate()) {
        return
      }
      if (this.customValidationMessage !== '') {
        return
      }
      this.listLoading = true
      let param = {
        repositoryId: this.repositoryId,
        id: this.attrId,
        name: this.attrName
      }
      this.editAttribute(param).then(() => {
        this.getItems()
      }).catch(() => {
        this.listLoading = false
      })
      this.closeSimpleDialog()
    },

    changeInput: function(e: { detail: unknown[] }): void {
      this.attrName = e.detail[0]
      this.customValidationMessage = ''
      if (this.attrName.length > 0){
        this.customValidationMessage = utils.customValidationMessage({
          name: this.attrName,
          hasSameTabName: false,
        })
      }
    },

    getItems: function (): void{
      this.getAttributeList({repositoryId: this.repositoryId, filter: encodeURIComponent(this.searchString)}).then(async (e) => {
        this.newItems = e.items
        this.newTotal = e.items.length
        this.getTotalStatus()
        this.getSelectableStatus()
        this.listLoading = false
      })
    },
    //
    getSelectableStatus: function (): void {
      this.isSelect = 'true'
      if (this.newItems.length == 0) {
        this.selectable = 'false'
        this.newItems.push({
          id: 0,
          name: this.$t('attributeManagement.notExist'),
        })
      } else {
        this.selectable = 'true'
      }
    },
    //很重要 get属性类型
    getLabelByValue: function (value: string): void {
      if (value){
        let item = this.typeList.filter((item)=>item.value == value)
        return item[0].label
      }
      return
    },
    //
    getTotalStatus: function (): void {
      this.getAttributeList({repositoryId: this.repositoryId}).then((e) => {
        this.itemTotal = e.items.length
        if (this.itemTotal > 199){
          this.addButton = 'true'
        } else {
          this.addButton = 'false'
        }
      })
    },
    //paixu
    sort(options: { key: string, order: string }): void {
      this.sortOption.key = options.key
      this.sortOption.order = options.order
      this.newItems.sort((a, b) => {
        if (a[options.key] !== undefined && b[options.key] !== undefined) {
          if (options.order === 'asc') {
            return a[options.key].localeCompare(b[options.key])
          } else {
            return b[options.key].localeCompare(a[options.key])
          }
        }
      })
    },
  },
}
</script>

<style scoped>
.service-template {
  height: auto;
  width: 100%;
}

.content-top {
  width: 100%;
  height: 40px;
  display: flex;
  flex-wrap: nowrap;
  z-index: 0;
}
.top-left {
  display: flex;
  flex-wrap: nowrap;
  float: left;
}

.top-remove{
  padding-top: 8px;
  padding-right: 8px;
  float: left;
  z-index: 0;
}

.top-search {
  padding-top: 8px;
  padding-left: 8px;
  float: left;
  z-index: 0;
}
.top-right {
  padding-top: 8px;
  margin-left: auto;
  z-index: 0;
}

.content-table {
  width: 100%;
  z-index: 0;
}

.ui-table-main {
  margin: 0px;
}

.ui-table-main {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
}

.ui-table-main .ui-table-content {
  position: relative;
}
.line-item{
  /*padding-bottom: 40px;*/
}
.item-label {
  padding-bottom: 16px;
}
.item-input {
  padding-bottom: 32px;
  float: left;
}
.text-overflow {
  overflow:hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  word-break: break-all;
}
</style>
