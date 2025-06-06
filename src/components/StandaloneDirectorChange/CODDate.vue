<template>
  <v-card
    flat
    class="cod-date-container"
  >
    <div class="meta-container">
      <label>Director Change Date</label>

      <div class="value date">
        <v-menu
          ref="menu"
          v-model="menu"
          :nudge-right="40"
          transition="scale-transition"
          offset-y
          min-width="18rem"
        >
          <template #activator="{ on, attrs }">
            <v-text-field
              id="cod-textfield"
              v-model="dateFormatted"
              data-test-id="cod-date-text"
              :rules="codDateRules"
              label="Enter your Director Change Date"
              hint="YYYY/MM/DD"
              append-icon="mdi-calendar"
              filled
              v-bind="attrs"
              v-on="on"
              @click:append="menu = true"
            />
          </template>
          <v-date-picker
            id="cod-datepicker"
            v-model="date"
            data-test-id="cod-date-picker"
            :min="minDate"
            :max="maxDate"
            no-title
            @input="menu=true"
          />
        </v-menu>

        <div
          v-if="$v.dateFormatted.isNotNull"
          class="validationErrorInfo"
          data-test-id="cod-validation-error"
        >
          <span v-if="!$v.dateFormatted.isValidFormat">
            Date must be in format YYYY/MM/DD.
          </span>
          <span v-else-if="!$v.dateFormatted.isValidCodDate">
            Please enter a day between {{ formatDate(minDate, false) }} and {{ formatDate(maxDate, false) }}.
          </span>
        </div>
      </div>
    </div>
  </v-card>
</template>

<script lang="ts">
import { Component, Emit, Mixins, Prop, Watch } from 'vue-property-decorator'
import { isNotNull, isValidFormat, isValidCodDate } from '@/validators'
import { Getter } from 'pinia-class'
import { DateMixin } from '@/mixins'
import { useBusinessStore, useRootStore } from '@/stores'

@Component({
  validations: {
    dateFormatted: { isNotNull, isValidFormat, isValidCodDate }
  }
})
export default class CodDate extends Mixins(DateMixin) {
  // Prop passed into this component.
  @Prop({ default: '' }) readonly initialCodDate!: string

  @Getter(useRootStore) getCurrentDate!: string
  @Getter(useBusinessStore) getFoundingDate!: Date
  @Getter(useBusinessStore) getLastAnnualReportDate!: string
  @Getter(useBusinessStore) getLastDirectorChangeDate!: string
  @Getter(useBusinessStore) isBaseCompany!: boolean

  // Local properties.
  date = '' // bound to date picker
  dateFormatted = '' // bound to text field
  menu = false // bound to calendar menu

  /** The array of validations rules for the COD Date text field. */
  get codDateRules (): Array<(v) => boolean | string> {
    return [
      v => isNotNull(v) || 'A Director change date is required.'
    ]
  }

  /** The maximum date that can be entered. */
  get maxDate (): string {
    return this.getCurrentDate
  }

  /** The minimum date that can be entered. */
  get minDate (): string {
    let date: string = null

    if (this.isBaseCompany) {
      // For BEN/BC/CC/ULC and CBEN/C/CCC/CUL, use the last COD filing in filing history.
      date = (this.getLastDirectorChangeDate || this.dateToYyyyMmDd(this.getFoundingDate))
    } else if (this.getLastDirectorChangeDate || this.getLastAnnualReportDate) {
      // For Coops, use the latest of the following dates:
      // - the last COD filing in filing history
      // - the last AR filing in filing history
      date = this.latestYyyyMmDd(this.getLastDirectorChangeDate, this.getLastAnnualReportDate)
    } else {
      // If the entity has no filing history then use the founding date.
      date = this.dateToYyyyMmDd(this.getFoundingDate)
    }

    return date
  }

  /** Called when component is mounted. */
  mounted (): void {
    // load initial data
    this.dateFormatted = this.formatDate(this.initialCodDate)
  }

  /**
   * Local helper to change date from YYYY-MM-DD to YYYY/MM/DD.
   * @returns The formatted date.
   */
  formatDate (date: string, validate = true): string {
    if (validate && !this.isValidDate(date, '-')) return ''
    const [year, month, day] = date.split('-')
    return `${year}/${month}/${day}`
  }

  /**
   * Local helper to change date from YYYY/MM/DD to YYYY-MM-DD.
   * @returns The parsed date.
   */
  parseDate (date: string): string {
    // changes date from YYYY/MM/DD to YYYY-MM-DD
    if (!this.isValidDate(date, '/')) return ''
    const [year, month, day] = date.split('/')
    return `${year}-${month}-${day}`
  }

  /**
   * Local helper to determine if passed-in date is valid.
   * @returns True if date is valid, otherwise false.
   */
  isValidDate (date, separator): boolean {
    return (isNotNull.call(this, date) &&
      isValidFormat.call(this, date, separator) &&
      isValidCodDate.call(this, date, separator))
  }

  /**
   * When prop changes, load (initial) data.
   */
  @Watch('initialCodDate')
  onInitialCodChanged (val: string): void {
    if (val) {
      this.dateFormatted = this.formatDate(val)
    }
  }

  /**
   * When text field changes, update date picker.
   */
  @Watch('dateFormatted')
  onDateFormattedChanged (val: string): void {
    this.date = this.parseDate(val)
  }

  /**
   * When date picker changes, update text field etc.
   */
  @Watch('date')
  onDateChanged (val: string): void {
    const codDate = this.isValidDate(val, '-') ? val : null
    // only update text field if date is valid
    // this is to retain previous invalid values
    if (codDate) {
      this.dateFormatted = this.formatDate(val)
    }
    this.emitCodDate(codDate)
    this.emitValid(Boolean(codDate))
  }

  /**
   * Emits an event with the new value of COD Date.
   */
  @Emit('codDate')
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  emitCodDate (val: string): void {}

  /**
   * Emits an event indicating whether or not this component is valid.
   */
  @Emit('valid')
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  emitValid (val: boolean): void {}
}
</script>

<style lang="scss" scoped>
.cod-date-container {
  padding: 1.25rem;
}

.validationErrorInfo {
  color: red;
}

.value.date {
  min-width: 24rem;
}

.meta-container {
  display: flex;
  flex-flow: column nowrap;
  position: relative;

  > label:first-child {
    font-weight: 700;
  }
}

@media (min-width: 768px) {
  .meta-container {
    flex-flow: row nowrap;

    > label:first-child {
      flex: 0 0 auto;
      padding-right: 2rem;
      width: 12rem;
    }
  }
}
</style>
