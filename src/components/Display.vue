<template>
  <div id="display">
    <v-card flat id="history-display" class="text-xs-left pl-2">
      <span v-for="item in history">
        {{ item.value }}<sub>{{ item.base }}</sub>{{ item.operation }}
      </span>
    </v-card>

    <v-card id="active-display" class="pr-4 amber lighten-4 text-xs-right">
      <span v-show="mode === 'hex'"> {{ formattedHex }} </span>
      <span v-show="mode === 'dec'"> {{ dec }} </span>
    </v-card>

    <v-card flat id="binary-display" class="text-xs-right pa-2 my-2">
      <template v-for="(bit, index) in formattedBinary">
        <span :class="index % 4 === 0 ? 'pl-3' : ''" @click="bitClick(bit, index)"> {{ bit }} </span>
      </template>
    </v-card>
  </div>
</template>


<script>
export default {
  name: 'display',
  props: ['dec', 'hex', 'binary', 'history', 'mode', 'bitSize'],
  computed: {
    formattedHex: function () {
      return this.hex.replace('0x', '').toUpperCase()
    },
    formattedBinary: function () {
      let binaryNoPrefix = this.binary.replace('0b', '')
      let padding = '0'.repeat(this.bitSize - binaryNoPrefix.length)
      return padding.concat(binaryNoPrefix)
    }
  },
  methods: {
    bitClick: function (bit, index) {
      // index here is relative to the string binary representation
      // we need to flip it around to represent the actual binary index
      let binaryIndex = this.bitSize - 1 - index
      this.$emit('binary-mod', bit, binaryIndex)
    }
  }
}
</script>

<style scoped>
#active-display {
  font-size: 2em;
  line-height: 2em;
}
#history-display {
  min-height: 1.5em;
}
sub {
  font-size: 0.5em;
}
#binary-display {
  letter-spacing: 2px;
  min-height: 2em;
}
</style>
