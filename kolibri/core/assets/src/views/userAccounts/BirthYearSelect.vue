<template>

  <div class="pos-rel">
    <KSelect
      class="birthyear-select"
      :value="selected"
      :label="coreString('birthYearLabel')"
      :placeholder="$tr('placeholder')"
      :options="options"
      :disabled="$attrs.disabled"
      @change="$emit('update:value', $event.value)"
    />
    <CoreInfoIcon
      class="info-icon"
      :tooltipText="$tr('birthYearTooltip')"
      :tooltipPlacement="tooltipPlacement"
      :iconAriaLabel="$tr('birthyearAriaLabel')"
    />

  </div>

</template>


<script>

  import range from 'lodash/range';
  import getYear from 'date-fns/get_year';
  import { now } from 'kolibri.utils.serverClock';
  import CoreInfoIcon from 'kolibri.coreVue.components.CoreInfoIcon';
  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import responsiveWindowMixin from 'kolibri.coreVue.mixins.responsiveWindowMixin';
  import { DemographicConstants } from 'kolibri.coreVue.vuex.constants';

  const { NOT_SPECIFIED } = DemographicConstants;

  // Take the last-known year to be the later of the copyright year,
  // or the year of the server date
  const firstYear = Math.max(Number(__copyrightYear), getYear(now()));

  function makeYearOptions(max, min) {
    return range(max, min, -1).map(n => ({
      label: String(n),
      value: String(n),
    }));
  }

  const yearOptions = makeYearOptions(firstYear, 1900);

  export default {
    name: 'BirthYearSelect',
    components: {
      CoreInfoIcon,
    },
    mixins: [commonCoreStrings, responsiveWindowMixin],
    props: {
      value: {
        type: String,
        default: null,
      },
    },
    computed: {
      selected() {
        return this.options.find(o => o.value === this.value) || {};
      },
      options() {
        // The backend validation actually lets you pick years up to 3000, so we'll
        // fill in the gaps just in case a user was given a later date, e.g. via CSV
        let extraYears = [];
        if (Number(this.value) > firstYear) {
          extraYears = makeYearOptions(Number(this.value), firstYear - 1);
        }
        return [
          {
            value: NOT_SPECIFIED,
            label: this.coreString('birthYearNotSpecified'),
          },
          ...extraYears,
          ...yearOptions,
        ];
      },
      tooltipPlacement() {
        if (this.windowIsSmall) {
          return 'left';
        }
        return 'bottom';
      },
    },
    $trs: {
      placeholder: 'Select year',
      birthYearTooltip: 'Provide an estimate if you are unsure.',
      birthyearAriaLabel: {
        message: 'About providing your birth year.',
        context:
          "\nCould also be translated as \"View information about providing your birth year\"\n\nAll 'AriaLabel' type of messages are providing additional context to the screen-reader users. \n\nIn this case the screen-reader will announce the message to indicate that the 'i' icon for the 'Birth year' field offers suggestions how to include that information when creating the user.",
      },
    },
  };

</script>


<style lang="scss" scoped>

  .pos-rel {
    position: relative;
  }

  .birthyear-select {
    width: calc(100% - 32px);
  }

  .info-icon {
    position: absolute;
    top: 27px;
    right: 0;
  }

</style>
