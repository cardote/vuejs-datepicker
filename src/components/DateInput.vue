<template>
  <div :class="{'input-group' : bootstrapStyling}">
    <!-- Calendar Button -->
    <span v-if="calendarButton" class="vdp-datepicker__calendar-button" :class="{'input-group-prepend' : bootstrapStyling}" @click="showCalendar" v-bind:style="{'cursor:not-allowed;' : disabled}">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <input
      :type="inline ? 'hidden' : 'text'"
      :class="computedInputClass"
      :name="name"
      :ref="refName"
      :id="id"
      :value="formattedValue"
      :open-date="openDate"
      :placeholder="placeholder"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"
      autocomplete="off">
    <!-- Clear Button -->
    <span v-if="clearButton && selectedDate" class="vdp-datepicker__clear-button" :class="{'input-group-append' : bootstrapStyling}" @click="clearDate()">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="clearButtonIcon">
          <span v-if="!clearButtonIcon">&times;</span>
        </i>
      </span>
    </span>
    <slot name="afterDateInput"></slot>
  </div>
</template>
<script>
import { makeDateUtils } from '../utils/DateUtils'
export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: String,
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      input: null,
      typedDate: false,
      utils: constructedDateUtils
    }
  },
  computed: {
    formattedValue () {
      if (!this.selectedDate) {
        return null
      }
      if (this.typedDate) {
        return this.typedDate
      }
      return typeof this.format === 'function'
        ? this.format(this.selectedDate)
        : this.utils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    },

    computedInputClass () {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === 'string') {
          return [this.inputClass, 'form-control'].join(' ')
        }
        return {'form-control': true, ...this.inputClass}
      }
      return this.inputClass
    }
  },
  watch: {
    resetTypedDate () {
      this.typedDate = false
    }
  },
  methods: {
    showCalendar () {
      this.$emit('showCalendar')
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate (event) {
      // close calendar if escape or enter are pressed
      if ([
        27, // escape
        13 // enter
      ].includes(event.keyCode)) {
        this.input.blur()
      }

      if (this.typeable) {
        const typedDate = Date.parse(this.input.value)
        var parseableDate = this.parseableDate(this.input.value, this.format)
        var parsedDate = Date.parse(parseableDate)
        if (!isNaN(parsedDate)) {
          this.typedDate = this.input.value
          this.$emit('typedDate', new Date(parsedDate))
        }
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred () {
      var parseableDate = this.parseableDate(this.input.value, this.format)
      if (isNaN(Date.parse(parseableDate))) {
        this.clearDate()
        this.input.value = null
        this.typedDate = null
      }

      this.$emit('closeCalendar')
    },
    /**
     * emit a clearDate event
     */
    clearDate () {
      this.$emit('clearDate')
    },
     /**
     * makes date parseable
     * to use with international dates
     */
    parseableDate (datestr, formatstr) {
      if (!(datestr && formatstr)) { return datestr }
      var splitter = formatstr.match(/-|\/|\s|\./) || ['-']
      var df = formatstr.split(splitter[0])
      var ds = datestr.split(splitter[0])
      var ymd = [0, 0, 0]
      var dat
      for (var i = 0; i < df.length; i++) {
        if (/yyyy/i.test(df[i])) {
          ymd[0] = ds[i]
        } else if (/mm/i.test(df[i])) {
          ymd[1] = ds[i]
        } else if (/m/i.test(df[i])) {
          ymd[1] = ds[i]  
        } else if (/dd/i.test(df[i])) {
          ymd[2] = ds[i]
        } else if (/d/i.test(df[i])) {
          ymd[2] = ds[i]
        }
      }
      var timezone = new Date().toString().split(' ')
      dat = ymd.join('-') + 'T00:00:00' + timezone[5].substr(3, 5)  //  include timezone to avoid wrong dates after parse
      if (isNaN(Date.parse(dat))) {
        return datestr
      }
      return dat
  },
  mounted () {
    this.input = this.$el.querySelector('input')
  }
}
// eslint-disable-next-line
;
</script>
