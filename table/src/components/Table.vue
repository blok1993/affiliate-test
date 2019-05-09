<template>
    <div>
      <div class="header">
        Table UI
      </div>

      <div v-if="loading" class="loading">
        Data is loading...
      </div>

      <div v-if="loadError" class="error">
        Error occurred while loading data. Try again later.
      </div>

      <div v-show="!loading && !loadError" id="main">
        <div class="controls">
          <div class="group-by">
            <span class="bold">Group By:</span>
            <div class="pills">
              <div class="pill"
                   v-for="col in gridColumns"
                   v-bind:class="{'active': selectedCol === col, 'disabled': selectedCol !== col && includedColumns.indexOf(col) === -1}"
                   @click="makeFirst(col)">{{col}}
              </div>
            </div>
          </div>

          <div class="delete-control">
            <button :disabled="checkedRows.length === 0"
                    class="red"
                    @click="deleteSelectedItems($event, checkedRows)">Delete
              <span v-if="checkedRows.length > 0">({{checkedRows.length}})</span>
            </button>
          </div>

          <div class="items-per-page">
            <multi-select
              v-model="perPageValue"
              :btnLabel="btnLabel"
              :selectOptions="perPageData"
              @selectionChanged="perPageChanged"/>
          </div>

          <div class="page-control">
            <button class="control control-left"
                    :disabled="page === 0"
                    @click="changePage(-1)"><
            </button>
            <span class="text">{{page * pageSize + 1}} - {{(page + 1) * pageSize}} of {{gridData.length}}</span>
            <button class="control control-right"
                    :disabled="page === Math.ceil(gridData.length / pageSize) - 1"
                    @click="changePage(1)">>
            </button>
          </div>

          <div class="columns-inclusion">
            <multi-select
              v-model="includedColumns"
              :btnLabel="inclusionBtnLabel"
              :options="options"
              :selectOptions="gridColumns"
              @selectionChanged="selectionChanged"/>
          </div>
        </div>

        <grid-template
          :ingredients="selectedGridData"
          :columns="copyOfGridColumns"
          @deleteItem="deleteItem"
          @checkedRow="rowChecked">
        </grid-template>
      </div>

      <Confirm v-show="showConfirm"
               :items="itemsForConfirmation"
               @ConfirmationConfirm="confirmAction"
               @ConfirmationReject="confirmReject">
      </Confirm>
    </div>
</template>

<script>

import GridTemplate from './GridTemplate.vue';
import Confirm from "./Confirm.vue";
import { emulateGetRequest, emulateDeleteRequest } from "../api";
import multiSelect from 'vue-multi-select';
import 'vue-multi-select/dist/lib/vue-multi-select.min.css';

export default {
  name: 'Table',
  components: {
    GridTemplate,
    multiSelect,
    Confirm
  },

  data() {
    return {
      loading: false,
      loadError: false,
      selectedCol: 'product',
      gridColumns: ['product', 'calories', 'fat', 'carbs', 'protein', 'iron'],
      copyOfGridColumns: ['product', 'calories', 'fat', 'carbs', 'protein', 'iron'],
      gridData: [],
      selectedGridData: [],
      page: 0,
      pageSize: 10,

      btnLabel: perPageValue => perPageValue.length > 0 ? perPageValue[0].name : 'Select count per page',
      perPageData: [
        { name: '5 Per Page', value: 5 },
        { name: '10 Per Page', value: 10 },
        { name: '25 Per Page', value: 25 },
        { name: '50 Per Page', value: 50 }
      ],
      perPageValue: [{ name: '10 Per Page', value: 10 }],

      inclusionBtnLabel: includedColumns => `${includedColumns.length} Columns Selected`,
      options: {
        multi: true,
        labelList: 'perPageData',
        cssSelected: option => (option.selected ? { 'background-color': '#4c73fc' } : ''),
        renderTemplate: elem => {
            let checkBox = `<label class="container">
                <input type="checkbox" ${this.includedColumns.indexOf(elem.name) > -1 ? 'checked="checked"' : ''}>
                <span class="checkmark"></span>
            </label>`;

            return `<div style="display: flex; pointer-events: none">${checkBox} ${elem.name}</div>`
        }
      },
      includedColumns: ['product', 'calories', 'fat', 'carbs', 'protein', 'iron'],

      checkedRows: [],

      showConfirm: false,
      itemsForConfirmation: []
    }
  },

  mounted() {
    this.loading = true;

    emulateGetRequest()
      .then(response => {
        this.loading = false;
        this.gridData = response;
        this.perPageChanged();
        this.makeFirst(this.selectedCol);
      })
      .catch(err => {
        console.log(err);
        this.loading = false;
        this.loadError = true;
      });
  },

  methods: {
      makeFirst(col) {
        if (this.includedColumns.indexOf(col) > -1) {
          this.selectedCol = col;
          this.copyOfGridColumns = [...this.gridColumns].filter(el => this.includedColumns.indexOf(el) > -1);
          this.copyOfGridColumns.splice(this.copyOfGridColumns.indexOf(col), 1);
          this.copyOfGridColumns.unshift(col);
        }
      },

      selectionChanged() {
        if (this.includedColumns.length > 0) {
          let localArr = [...this.includedColumns];
          this.copyOfGridColumns = localArr;

          if (!localArr.includes(this.selectedCol)) {
            this.copyOfGridColumns.unshift(this.selectedCol);
          } else {
            this.copyOfGridColumns.splice(this.copyOfGridColumns.indexOf(this.selectedCol), 1);
            this.copyOfGridColumns.unshift(this.selectedCol);
          }
        } else {
          this.includedColumns = [...this.gridColumns];
        }
      },

      perPageChanged() {
        if (this.perPageValue.length > 0) {
          this.page = 0;
          this.pageSize = this.perPageValue[0].value;
          this.selectedGridData = [...this.gridData].splice(0, this.pageSize);
        }
      },

      changePage(direction) {
        this.page += direction;
        this.selectedGridData = [...this.gridData].splice(this.page * this.pageSize, this.pageSize);
      },

      rowChecked(e) {
          this.checkedRows = Array.from(e);
      },

      deleteSelectedItems(e, entries) {
        this.showConfirm = true;
        this.itemsForConfirmation = [...entries];

        let rect = e.target.getBoundingClientRect();
        let popup = document.querySelector('.confirm-popup');
        popup.style.left = `${rect.left - 200 > 0 ? rect.left - 200 : 0}px`;
        popup.style.top = `${rect.top + rect.height + window.pageYOffset + 10}px`;
      },

      deleteItem(arr) {
        this.deleteSelectedItems(arr[0], arr[1]);
      },

      confirmAction(items) {
        this.showConfirm = false;

        // items are needed there to delete them
        emulateDeleteRequest()
          .then(response => {
            console.log('Items deleted successfully.');
          })
          .catch(error => {
            console.error('Error occurred while deleting items.');
          });
      },

      confirmReject() {
        this.showConfirm = false;
      }
  }
}
</script>

<style lang="scss">
  @import '../assets/scss/table';
</style>
