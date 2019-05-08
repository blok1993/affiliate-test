<template>
  <table>
    <thead>
      <tr>
        <th>
          <input type="checkbox" />
        </th>
        <th v-for="key in columns"
          @click="sortBy(key)"
          :class="{ active: sortKey === key }">
          {{ key | capitalize }}
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
          <input type="checkbox" />
        </td>
        <td v-for="key in columns">
          {{entry[key]}}
        </td>
        <td>
          <div class="delete red" @click="deleteItem(entry)">Delete</div>
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

    data () {
      const sortOrders = {};
      this.columns.forEach(key => {
        sortOrders[key] = 1
      });

      return {
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

      deleteItem(entry) {
        emulateDeleteRequest()
          .then(response => {
              console.log('Item deleted successfully.');
          })
          .catch(error => {
              console.error('Error occurred while deleting item.');
          });
      }
    }
  }
</script>

<style scoped>

</style>
