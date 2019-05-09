<template>
  <div class="confirm-popup">
    <div class="title"
          v-html="title">
    </div>
    <div class="buttons-block">
      <button @click="reject">Cancel</button>
      <button class="red" @click="confirmation">Confirm</button>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Confirm',
    props: {
      title: {
        type: String,
        default: 'Are you sure you want to <span class="bold">delete item</span>?'
      },
      items: {
        type: Array,
        default: () => []
      }
    },

    methods: {
      confirmation() {
        this.$emit('ConfirmationConfirm', this.items);
      },

      reject() {
        this.$emit('ConfirmationReject');
      }
    }
  }
</script>
