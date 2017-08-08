<template>
  <v-btn class="amber lighten-2"
  :dark="active" :outline="!active" :disabled="!active"
  @click="handleClick">
    {{ value }}
  </v-btn>
</template>


<script>
export default {
  name: 'digit-button',
  methods: {
    handleClick: function (event) {
      this.$emit('digit-click', this.value)
    }
  },
  props: {
    value: {
      type: [String, Number],
      required: true
    },
    active: {
      default: true,
      type: Boolean
    }
  }
}
</script>

<style scoped>
button {
  font-size: 1.2em;
}
</style>
