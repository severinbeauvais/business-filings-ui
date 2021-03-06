<template>
  <div v-if="filing" class="future-effective-details body-2">
    <h4>{{_.subtitle}}</h4>

    <p>The {{_.filingLabel}} date and time for {{_.companyLabel}}
      will be <strong>{{effectiveDateTime}} Pacific Time</strong>.</p>

    <p>If you wish to change the information in this {{_.filingLabel}}, you must
      contact Registry Staff to file a withdrawal.</p>

    <p>Withdrawing this {{_.filingTitle}} will remove this {{_.filingLabel}}
      and all associated information, and will incur a $20.00 fee.</p>

    <p v-if="filing.courtOrderNumber">Court Order Number: {{filing.courtOrderNumber}}</p>

    <p v-if="filing.isArrangement">Pursuant to a Plan of Arrangement</p>

    <p>Registries contact information:</p>

    <contact-info class="pt-3" />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
import { mapState } from 'vuex'
import { ContactInfo } from '@/components/common'
import { HistoryItemIF } from '@/interfaces'

@Component({
  computed: { ...mapState(['entityName']) },
  components: { ContactInfo }
})
export default class FutureEffective extends Vue {
  readonly entityName!: string

  /** The subject filing. */
  @Prop({ required: true }) private filing: HistoryItemIF

  /** Data for the subject filing. */
  private get _ (): any {
    if (this.filing.isFutureEffectiveIa) {
      return {
        subtitle: 'Future Effective Incorporation Date',
        filingLabel: 'incorporation',
        companyLabel: (this.entityName || 'this Numbered Benefit Company'),
        filingTitle: 'Incorporation Application'
      }
    }
    if (this.filing.isFutureEffectiveAlteration) {
      return {
        subtitle: 'Future Effective Alteration Date',
        filingLabel: 'alteration',
        companyLabel: (this.entityName || 'this company'),
        filingTitle: 'Alteration Notice'
      }
    }
    return {
      subtitle: 'Future Effective Filing Date',
      filingLabel: 'filing',
      companyLabel: (this.entityName || 'this company'),
      filingTitle: 'Filing'
    }
  }

  /** The future effective datetime of the subject filing. */
  private get effectiveDateTime (): string {
    return this.filing.effectiveDateTime || 'unknown'
  }
}
</script>

<style lang="scss" scoped>
h4 {
  letter-spacing: 0;
  font-size: 0.9375rem;
  font-weight: 700;
}

p:first-of-type,
p:last-of-type {
  padding-top: 0.75rem;
}

p {
  margin-bottom: 0.5rem !important;
}
</style>
