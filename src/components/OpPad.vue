<template>
  <div id="op-pad">
    <v-layout row>
      <v-layout column>
        <v-btn class="pa-1" :key="op.id" v-for="op in arithmetic" @click="handleClick(op, false)"> {{ op }} </v-btn>
      </v-layout>
      <v-layout column>
        <v-btn class='pa-1 red accent-1' dark raised @click="handleClear(false)"> C </v-btn>
        <v-btn class="pa-1" :key="op.id" v-for="op in bitwise" @click="handleClick(op, false)"> {{ op }} </v-btn>
      </v-layout>
      <v-layout column>
        <v-btn class='pa-1 red accent-2' dark raised @click="handleClear(true)"> CE </v-btn>
        <v-btn class="pa-1" :key="op.id" v-for="op in unary" @click="handleClick(op, true)"> {{ op }} </v-btn>
      </v-layout>
    </v-layout>
  </div>
</template>


<script>
import math from 'mathjs'

export default {
  name: 'op-pad',
  props: ['leftOperand', 'rightOperand'],
  methods: {
    handleClear: function (clearAll) {
      this.$emit('clear', clearAll)
    },
    handleClick: function (operation, isUnary) {
      let result = null
      let intermediateOperand = null
      if (this.leftOperand) {
        result = this.applyBinaryOp()
        intermediateOperand = result
      }

      this.activeOperation = operation

      if (isUnary) {
        result = this.applyUnaryOp(intermediateOperand)
      }

      this.$emit('op-click', operation, result, isUnary)
    },
    applyUnaryOp: function (value) {
      let computed
      let operand = value || this.rightOperand
      switch (this.activeOperation) {
        case '<<':
          computed = math.leftShift(operand, 1)
          break
        case '>>':
          computed = math.rightArithShift(operand, 1)
          break
        case '=':
          computed = operand
          break
        case 'NOT':
          computed = math.bitNot(operand)
          break
      }
      return computed
    },
    applyBinaryOp: function () {
      let computed
      switch (this.activeOperation) {
        case '+':
          computed = math.add(this.leftOperand, this.rightOperand)
          break
        case '-':
          computed = math.subtract(this.leftOperand, this.rightOperand)
          break
        case '*':
          computed = math.multiply(this.leftOperand, this.rightOperand)
          break
        case '/':
          computed = math.floor(math.divide(this.leftOperand / this.rightOperand))
          break
        case '%':
          computed = math.mod(this.leftOperand, this.rightOperand)
          break
        case 'AND':
          computed = math.bitAnd(this.leftOperand, this.rightOperand)
          break
        case 'OR':
          computed = math.bitOr(this.leftOperand, this.rightOperand)
          break
        case 'XOR':
          computed = math.bitXor(this.leftOperand, this.rightOperand)
          break
        case 'X<<Y':
          computed = math.leftShift(this.leftOperand, this.rightOperand)
          break
        case 'X>>Y':
          computed = math.rightArithShift(this.leftOperand, this.rightOperand)
          break
      }
      return computed
    }
  },
  data () {
    return {
      activeOperation: '',
      arithmetic: ['+', '-', '/', '*', '%'],
      bitwise: ['AND', 'OR', 'XOR', 'X<<Y', 'X>>Y'],
      unary: ['>>', '<<', 'NOT', '=']
    }
  }
}
</script>

<style scoped>
</style>
