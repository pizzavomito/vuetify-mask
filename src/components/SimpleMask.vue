<template>
  <div>
    <v-text-field
      v-model="cmpValue"
      v-bind:label="label"
      v-bind="properties",
      v-on:click:append-outer="$emit('click:append-outer')"
      v-on:click:append="$emit('click:append')"
      v-on:click:prepend="$emit('click:preprend')"
      v-on:click:prepend-inner="$emit('click:preprend-inner')"
      v-on:click:clear="$emit('click:clear')"
      v-bind:maxlength="options.inputMask.length"
      v-on:keypress="keyPress"
      v-on:blur="$emit('blur')"
      v-on:change="$emit('change')"
      v-on:click="$emit('click')"
      v-on:focus="$emit('focus')"
      v-on:keydown="$emit('keydown')"
      v-on:mousedown="$emit('mousedown')"
      v-on:mouseup="$emit('mouseup')"
      ref="ref"
    ></v-text-field>
  </div>
</template>

<script>
export default {
  model: { prop: "value", event: "input" },
  props: {
    value: {
      type: [String, Number],
      default: "0",
    },
    label: {
      type: String,
      default: "",
    },
    properties: {
      type: Object,
      default: function() {
        return {};
      },
    },
    options: {
      type: Object,
      default: function() {
        return {
          inputMask: "#########",
          outputMask: "#########",
          empty: "", // v-model value when v-text-field is empty. You can use "0" or "" or null or other.
          applyAfter: true, // Apply the mask only after filling
          alphanumeric: false,
          lowerCase: false,
        };
      },
    },
  },
  data: () => ({}),
  /*
   v-model="cmpValue": Dessa forma, ao digitar, o valor é atualizado automaticamente no componente pai.
   O valor digitado entra pelo newValue do Set é emitido para o componente pai, retorna pelo get e pára.
  */
  computed: {
    cmpValue: {
      get: function() {
        return this.humanFormat(this.value);
      },
      set: function(newValue) {
        this.$emit("input", this.machineFormat(newValue));
      },
    },
  },
  watch: {},
  methods: {
    humanFormat: function(value) {
      if (value) {
        value = this.formatValue(value, this.options.inputMask);
      } else {
        value = this.options.empty;
      }
      return value;
    },

    machineFormat(value) {
      if (value) {
        value = this.formatValue(value, this.options.outputMask);
        if (value === "") {
          value = this.options.empty;
        }
        // UpperCase or LowerCase
        if (this.options.lowerCase) {
          value = value ? value.toLowerCase() : value;
        } else {
          value = value ? value.toUpperCase() : value;
        }
        // Apply the mask only after filling
        if (this.options.applyAfter) {
          if (value.length !== this.options.outputMask.length) {
            value = this.options.empty;
          } else {
            // Event sended after filling the mask
            this.$emit("masked");
          }
        }
      } else {
        value = this.options.empty;
      }
      return value;
    },

    formatValue: function(value, mask) {
      return this.formatDefault(value, mask);
    },

    formatDefault: function(value, mask) {
      value = this.clearValue(value);
      let result = "";
      let count = 0;
      if (value) {
        let arrayValue = value.toString().split("");
        let arrayMask = mask.toString().split("");
        for (var i = 0; i < arrayMask.length; i++) {
          if (i < arrayValue.length + count) {
            if (arrayMask[i] === "#") {
              result = result + arrayValue[i - count];
            } else {
              result = result + arrayMask[i];
              count++;
            }
          }
        }
      }
      return result;
    },

    keyPress($event) {
      if (!this.options.alphanumeric) {
        // console.log($event.keyCode); //keyCodes value
        let keyCode = $event.keyCode ? $event.keyCode : $event.which;
        // if ((keyCode < 48 || keyCode > 57) && keyCode !== 46) {
        if (keyCode < 48 || keyCode > 57) {
          // 46 is dot
          $event.preventDefault();
        }
      }
    },

    clearValue: function(value) {
      let result = "";
      if (value) {
        let arrayValue = value.toString().split("");
        for (var i = 0; i < arrayValue.length; i++) {
          if (this.isValid(arrayValue[i])) {
            result = result + arrayValue[i];
          }
        }
      }
      return result;
    },

    isValid(value) {
      if (this.options.alphanumeric) {
        return this.isAlphaNumeric(value);
      } else {
        return this.isInteger(value);
      }
    },

    isInteger(value) {
      let result = false;
      if (Number.isInteger(parseInt(value))) {
        result = true;
      }
      return result;
    },

    isAlphaNumeric(value) {
      let letterNumber = /^[0-9a-zA-Z]+$/;
      if (value.match(letterNumber)) {
        return true;
      }
      return false;
    },

    focus() {
      setTimeout(() => {
        this.$refs.ref.focus();
      }, 500);
    },
    
  },
};
</script>
