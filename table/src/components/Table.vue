<template>
    <div id="demo">
      <div class="controls">
        <div class="group-by">
          <b>Group By:</b>
          <div class="pills">
            <div class="pill"
                 v-for="col in gridColumns"
                 v-bind:class="{'active': selectedCol === col}"
                 @click="makeFirst(col)">{{col}}
            </div>
          </div>
        </div>
      </div>

      <grid-template
        :heroes="gridData"
        :columns="gridColumns"
        :filter-key="searchQuery">
      </grid-template>
    </div>
</template>

<script>
import GridTemplate from './GridTemplate.vue';

export default {
  name: 'Table',
  components: {
    GridTemplate
  },

  data() {
    return {
      searchQuery: '',
      selectedCol: '',
      gridColumns: ['name', 'power'],
      gridData: [
        { name: 'Chuck Norris', power: Infinity },
        { name: 'Bruce Lee', power: 9000 },
        { name: 'Jackie Chan', power: 7000 },
        { name: 'Jet Li', power: 8000 }
      ]
    }
  },

  methods: {
      makeFirst: function(col) {
          this.gridColumns.splice(this.gridColumns.indexOf(col), 1);
          this.gridColumns.unshift(col);
      }
  }
}
</script>
