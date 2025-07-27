<template>
  <v-card :class="['mx-auto my-4 box-shadow-none license-plate-card', plateTypeClass]" :width="'fit-content'">
    <v-row dense :class="['license-plate-container no-gutters justify-flex-start', plateTypeClass]" align="center" justify="center">
      <!-- Last two digits -->
      <v-col cols="2" class="center-content last-two-digits">
        <span v-if="isReadOnly" class="text-display">
          {{ plateForm.digitsLast }}
        </span>
        <v-text-field
          v-else
          ref="digitsLast"
          dense
          hide-details
          type="tel"
          maxlength="2"
          v-model="plateForm.digitsLast"
          placeholder="99"
          class="ltr-item small-input font_aa px-md-0"
          :rules="[(val) => validateDigits(val, 2)]"
          @input="onInputDigitsLast"
        />
      </v-col>
      <!-- Middle three digits -->
      <v-col cols="3" class="center-content three-digits">
        <span v-if="isReadOnly" class="text-display">
          {{ plateForm.digitsMiddle }}
        </span>
        <v-text-field
          v-else
          ref="digitsMiddle"
          dense
          hide-details
          type="tel"
          maxlength="3"
          v-model="plateForm.digitsMiddle"
          placeholder="999"
          class="ltr-item small-input font_bold px-1"
          :rules="[(val) => validateDigits(val, 3)]"
          @input="onInputDigitsMiddle"
        />
      </v-col>
      <!-- Plate letter -->
      <v-col cols="3" class="center-content plate-letter">
        <span v-if="isReadOnly" class="text-display">
          {{ plateForm.letter }}
        </span>
        <v-select
          v-else
          ref="letter"
          :class="['letter-select', plateTypeClass]"
          v-model="plateForm.letter"
          :items="plateLetters"
          item-text="text"
          item-value="value"
          dense
          outlined
          hide-details
          :style="{ backgroundColor: '#AAA0f0' }"
          solo
          label="Letter"
          :filter="filterLetters"
          append-icon="mdi-chevron-down"
          @change="onSelectLetter"
        />
      </v-col>
      <!-- First two digits -->
      <v-col cols="2" class="center-content first-two-digits">
        <span v-if="isReadOnly" class="text-display">
          {{ plateForm.digitsFirst }}
        </span>
        <v-text-field
          v-else
          ref="digitsFirst"
          dense
          hide-details
          type="tel"
          maxlength="2"
          v-model="plateForm.digitsFirst"
          placeholder="99"
          class="ltr-item small-input font_bold px-1"
          :rules="[(val) => validateDigits(val, 2)]"
          @input="onInputDigitsFirst"
        />
      </v-col>
      <!-- Iran emblem -->
      <v-col class="plate-0">
        <img src="/pelak.svg" width="auto" height="100%" />
      </v-col>
    </v-row>
  </v-card>
</template>

<script>
export default {
  name: 'IranianLicensePlate',
  props: {
    value: { type: String, default: '' },
    readOnly: { type: Boolean, default: false },
    formatOutput: { type: Function, default: null }
  },
  data() {
    return {
      plateForm: {
        digitsFirst: '',
        letter: 'الف',
        digitsMiddle: '',
        digitsLast: '',
      },
      plateLetters: [
        { text: 'الف', value: 'الف' }, { text: 'ب', value: 'ب' },
        { text: 'ج', value: 'ج' }, { text: 'د', value: 'د' },
        { text: 'س', value: 'س' }, { text: 'ص', value: 'ص' },
        { text: 'ط', value: 'ط' }, { text: 'ق', value: 'ق' },
        { text: 'ل', value: 'ل' }, { text: 'م', value: 'م' },
        { text: 'ن', value: 'ن' }, { text: 'و', value: 'و' },
        { text: 'ه', value: 'ه' }, { text: 'ی', value: 'ی' },
        { text: 'ر', value: 'ر' }, { text: 'ک', value: 'ک' },
        { text: 'پ', value: 'پ' }, { text: 'ز', value: 'ز' },
        { text: 'ش', value: 'ش' }, { text: 'ث', value: 'ث' },
        { text: 'ع', value: 'ع' }, { text: 'ف', value: 'ف' },
        { text: 'گ', value: 'گ' }, { text: 'D', value: 'D' },
        { text: 'S', value: 'S' }
      ],
    }
  },
  computed: {
    plateTypeClass() {
      switch (this.plateForm.letter) {
        case 'ع': return 'plate-public'
        case 'ف': return 'plate-military'
        case 'گ': return 'plate-temporary'
        case 'D':
        case 'S': return 'plate-diplomatic'
        default: return 'plate-private'
      }
    },
    isReadOnly() { return this.readOnly },
  },
  watch: {
    plateForm: {
      deep: true,
      handler(newVal) {
        if (this.isReadOnly) return
        const { digitsFirst, letter, digitsMiddle, digitsLast } = newVal
        const formatted = this.formatOutput
          ? this.formatOutput({ digitsFirst, letter, digitsMiddle, digitsLast })
          : `${digitsFirst}-${letter}-${digitsMiddle}-${digitsLast}`
        this.$emit('input', formatted)
        this.$emit('change', { ...newVal, formatted })
      }
    },
    value(newVal) {
      this.parsePlateValue(newVal)
    }
  },
  mounted() {
    this.parsePlateValue(this.value)
  },
  methods: {
    parsePlateValue(plateString) {
      if (!plateString || typeof plateString !== 'string') return
      const parts = plateString.split('-')
      this.plateForm.digitsFirst = parts[0] || ''
      this.plateForm.letter = parts[1] || 'الف'
      this.plateForm.digitsMiddle = parts[2] || ''
      this.plateForm.digitsLast = parts[3] || ''
    },
    filterLetters(item, queryText) {
      return item.text.includes(queryText)
    },
    onInputDigitsLast() {
      if (this.plateForm.digitsLast.length === 2) {
        this.$refs.digitsMiddle && this.$refs.digitsMiddle.$el.querySelector('input')?.focus()
      }
    },
    onInputDigitsMiddle() {
      if (this.plateForm.digitsMiddle.length === 3) {
        this.$refs.letter && this.$refs.letter.$el.querySelector('input')?.focus()
      }
    },
    onSelectLetter() {
      this.$refs.digitsFirst && this.$refs.digitsFirst.$el.querySelector('input')?.focus()
    },
    onInputDigitsFirst() {},
    validateDigits(val, len) {
      return !val || (/^\d+$/.test(val) && val.length === len) || `Enter ${len} digits`
    }
  }
}
</script>

<style scoped>
.license-plate-card {
  border-radius: 6px;
  overflow: hidden;
  border: 2px solid #000;
  max-width: 350px;
}
.license-plate-container {
  background-color: #fff;
  border: 2px solid #000;
  border-radius: 4px;
  height: 50px;
  margin: 0 auto;
  padding: 0;
}
.license-plate-container > .v-col {
  padding: 0 !important;
  margin: 0 !important;
}
.center-content {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 4px;
}
.small-input {
  text-align: center;
  font-size: 1.2rem;
}
.iran-emblem-container {
  background-color: #039;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}
.last-two-digits {
  border-left: 2px solid #000;
}
.text-display {
  font-family: 'Arial', sans-serif;
  font-size: 1.3rem;
  font-weight: 700;
  user-select: none;
  color: inherit;
}
.plate-private { background-color: #ffffff; color: #000; }
.plate-public { background-color: #fdd835 !important; color: #000; }
.plate-public .letter-select >>> .v-input__control,
.plate-public .letter-select >>> .v-select__selections,
.plate-public .letter-select >>> .v-input__slot { background-color: #fdd835 !important; border: 1px solid #fdd835 !important; border-radius: 4px; }
.plate-military { background-color: #388e3c; color: #fff; }
.plate-military .letter-select >>> .v-input__control,
.plate-military .letter-select >>> .v-select__selections,
.plate-military .letter-select >>> .v-input__slot { background-color: #388e3c !important; border: 1px solid #388e3c !important; border-radius: 4px; }
.plate-diplomatic { background-color: #b71c1c; color: #fff; }
.plate-diplomatic .letter-select >>> .v-input__control,
.plate-diplomatic .letter-select >>> .v-select__selections,
.plate-diplomatic .letter-select >>> .v-input__slot { background-color: #b71c1c !important; border: 1px solid #b71c1c !important; border-radius: 4px; }
.plate-temporary { background-color: #f8e1e1; color: #000; }
.plate-temporary .letter-select >>> .v-input__control,
.plate-temporary .letter-select >>> .v-select__selections,
.plate-temporary .letter-select >>> .v-input__slot { background-color: #f8e1e1 !important; border: 1px solid #f8e1e1 !important; border-radius: 4px; }
.plate-0 {
  background: #039;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding: 0 !important;
  margin: 0 !important;
  box-sizing: border-box;
  place-content: center;
}
.plate-0 img {
  display: block;
  height: 100%;
  margin: 0 !important;
  padding: 0 !important;
  border: none;
}
.letter-select {
  min-width: 70px;
  max-width: 90px;
}
</style>
