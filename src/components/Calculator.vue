<template>
    <v-container id="calculator">

      <v-layout>
        <v-flex xs6 offset-xs3>
          <display :mode="mode" :history="history" :bitSize="size"
          :hex="hexValue" :dec="formattedDecValue" :binary="binaryValue"
          @binary-mod="handleBitToggle"></display>
          <v-toolbar dense flat>
            <v-btn-toggle mandatory :items="mode_options" v-model="mode"></v-btn-toggle>
            <v-btn-toggle mandatory :items="sign_options" v-model="signed"></v-btn-toggle>
            <v-btn-toggle mandatory :items="size_options" v-model="size"></v-btn-toggle>
          </v-toolbar>
        </v-flex>
      </v-layout>

      <v-layout justify-center class="mt-3">
        <v-flex xs3>
          <template v-for="n in ['D', 'E', 'F', 'A', 'B', 'C']">
            <digit-button :active="mode === 'hex'" :value="n" @digit-click="handleDigit"></digit-button>
          </template>

          <template v-for="n in [7,8,9,4,5,6,1,2,3,0]">
            <digit-button :value="n" @digit-click="handleDigit"></digit-button>
          </template>
        </v-flex>
        <v-flex xs3>
          <op-pad :leftOperand="leftOperand" :rightOperand="decValue"
          @clear="handleClear" @op-click="handleOp"> </op-pad>
        </v-flex>
      </v-layout>

    </v-container>
</template>

<script>
import DigitButton from './DigitButton'
import Display from './Display'
import OpPad from './OpPad'
import math from 'mathjs'

export default {
  name: 'calculator',
  components: {DigitButton, Display, OpPad},
  methods: {
    adjustForSizeAndSign: function () {
      let adjustedValue = this.decValue
      // first trim any bits out of the current size range
      let sizeMask = math.subtract(math.leftShift(math.bignumber(1), this.size), 1)
      adjustedValue = math.bitAnd(adjustedValue, sizeMask)

      if (this.signed) {
        // if we are treating this as signed, we want to preserve the binary
        // representation when adjusting the computed value
        this.signedBinary = adjustedValue.toBinary()
        // determine if the sign bit is 0 or 1
        let signMask = math.leftShift(math.bignumber(1), this.size - 1)
        let negative = !math.isZero(math.bitAnd(adjustedValue, signMask))

        if (negative) {
          // take 2s complement to determine absolute value
          adjustedValue = math.add(math.bitNot(adjustedValue), 1)
          // trim to size again, since the above operation would have flipped
          // all bits and we are pretending some of them don't exist
          adjustedValue = math.bitAnd(adjustedValue, sizeMask)
          adjustedValue = math.unaryMinus(adjustedValue)
        }
      }

      this.decValue = adjustedValue
    },
    handleBitToggle: function (bit, index) {
      if (bit === '0') {
        this.decValue = math.add(this.decValue, 2 ** index)
      } else {
        this.decValue = math.subtract(this.decValue, 2 ** index)
      }
      this.adjustForSizeAndSign()
    },
    handleDigit: function (digit) {
      let currentValue
      if (math.isZero(this.decValue) || !this.appendDigits) {
        currentValue = this.mode === 'dec' ? digit : `0x${digit}`
        this.appendDigits = true
      } else {
        currentValue = this.mode === 'dec' ? `${this.formattedDecValue}${digit}` : `${this.hexValue}${digit}`
      }
      this.decValue = math.bignumber(currentValue)
      this.adjustForSizeAndSign()
    },
    handleOp: function (operation, result, isUnary) {
      this.appendDigits = false
      this.history.push(
        {'value': this.mode === 'hex' ? this.hexValue : this.formattedDecValue,
          'base': this.base,
          'operation': operation
        })

      if (result) {
        this.decValue = result
        this.adjustForSizeAndSign()
      }

      if (isUnary) {
        this.history = []
        this.leftOperand = null
        return
      }

      this.leftOperand = this.decValue
    },
    handleClear: function (clearAll) {
      this.decValue = math.bignumber(0)
      this.signedBinary = '0b0'
      if (clearAll) {
        this.history = []
        this.leftOperand = null
      }
    }
  },
  watch: {
    mode: function () {
      this.appendDigits = false
    },
    size: function () {
      this.adjustForSizeAndSign()
    },
    signed: function () {
      this.adjustForSizeAndSign()
    }
  },
  computed: {
    base: function () {
      return this.mode === 'dec' ? 10 : 16
    },
    hexValue: function () {
      return this.decValue.toHexadecimal()
    },
    binaryValue: function () {
      return this.signed ? this.signedBinary : this.decValue.toBinary()
    },
    formattedDecValue: function () {
      return math.format(this.decValue, {notation: 'fixed'})
    }
  },
  data () {
    return {
      decValue: math.bignumber(0),
      leftOperand: null,
      appendDigits: true,
      history: [],
      signedBinary: '0b0',
      signed: true,
      sign_options: [
        {text: 'signed', value: true},
        {text: 'unsigned', value: false}
      ],
      size: 16,
      size_options: [
        {text: '8', value: 8},
        {text: '16', value: 16},
        {text: '32', value: 32},
        {text: '64', value: 64},
        {text: '128', value: 128}
      ],
      mode: 'dec',
      mode_options: [
        { text: 'hex', value: 'hex' },
        { text: 'dec', value: 'dec' }
      ]
    }
  }
}
</script>

<style scoped>
</style>
