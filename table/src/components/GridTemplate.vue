<template>
  <table>
    <thead>
      <tr>
        <th v-for="key in columns"
          @click="sortBy(key)"
          :class="{ active: sortKey == key }">
          {{ key | capitalize }}
          <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'">
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="entry in filteredHeroes">
        <td v-for="key in columns">
          {{entry[key]}}
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
  export default {
    name: 'GridTemplate',
    props: {
      heroes: {
        type: Array,
        default: () => []
      },
      columns:  {
        type: Array,
        default: () => []
      },
      filterKey: {
        type: String,
        default: ''
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
      filteredHeroes: function () {
        let sortKey = this.sortKey;
        let filterKey = this.filterKey && this.filterKey.toLowerCase();
        let order = this.sortOrders[sortKey] || 1;
        let heroes = this.heroes;

        if (filterKey) {
          heroes = heroes.filter(row => {
            return Object.keys(row).some(key => {
              return String(row[key]).toLowerCase().indexOf(filterKey) > -1
            })
          })
        }

        if (sortKey) {
          heroes = heroes.slice().sort((a, b) => {
            a = a[sortKey];
            b = b[sortKey];
            return (a === b ? 0 : a > b ? 1 : -1) * order;
          })
        }

        return heroes;
      }
    },

    filters: {
      capitalize: str => {
        return str.charAt(0).toUpperCase() + str.slice(1);
      }
    },

    methods: {
      sortBy: function(key) {
        this.sortKey = key;
        this.sortOrders[key] = this.sortOrders[key] * -1;
      }
    }
  }
</script>
