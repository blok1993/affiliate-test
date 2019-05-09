<template>
  <table>
    <thead>
      <tr>
        <th></th>
        <th v-for="key in columns"
          @click="sortBy(key)"
          :class="{ active: sortKey === key }">
          {{ key | capitalize }} {{measure[key]}}
          <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'">
          </span>
        </th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="entry in filteredIngredients" :key="entry.id">
        <td>
          <div class="vertical-line"></div>
          <label class="container">
            <input :id="entry.id" type="checkbox" :value="entry" @change="toggleRowEntry" v-model="checkedRows">
            <span class="checkmark"></span>
          </label>
        </td>
        <td v-for="key in columns">
          {{entry[key]}}
        </td>
        <td>
          <div class="delete red" @click="deleteItem($event, entry)">
            <div class="icon trash-icon"></div>
            Delete
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
  import { emulateDeleteRequest } from "../api";

  export default {
    name: 'GridTemplate',

    props: {
      ingredients: {
        type: Array,
        default: () => []
      },
      columns:  {
        type: Array,
        default: () => []
      }
    },

    data() {
      const sortOrders = {};
      this.columns.forEach(key => {
        sortOrders[key] = 1
      });

      return {
        measure: {
          'product': '(100g serving)',
          'calories': '',
          'carbs': '(g)',
          'fat': '(g)',
          'protein': '(g)',
          'iron': '(%)'
        },
        checkedRows: [],
        sortKey: '',
        sortOrders: sortOrders
      }
    },

    computed: {
      filteredIngredients() {
        let sortKey = this.sortKey;
        let order = this.sortOrders[sortKey] || 1;
        let ingredients = this.ingredients;

        if (sortKey) {
          ingredients = ingredients.slice().sort((a, b) => {
            a = a[sortKey];
            b = b[sortKey];
            return (a === b ? 0 : a > b ? 1 : -1) * order;
          })
        }

        return ingredients;
      }
    },

    filters: {
      capitalize: str => {
        return str.charAt(0).toUpperCase() + str.slice(1);
      }
    },

    methods: {
      sortBy(key) {
        this.sortKey = key;
        this.sortOrders[key] = this.sortOrders[key] * -1;
      },

      deleteItem(e, entry) {
        this.$emit('deleteItem', [e, entry]);
      },

      toggleRowEntry() {
        this.$emit('checkedRow', this.checkedRows);
      }
    }
  }
</script>

<style scoped>

</style>
