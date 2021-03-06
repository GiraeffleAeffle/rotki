<template>
  <v-container class="general-settings">
    <v-row no-gutters>
      <v-col>
        <v-card>
          <v-card-title>{{ $t('general_settings.title') }}</v-card-title>
          <v-card-text>
            <v-switch
              v-model="anonymizedLogs"
              class="general-settings__fields__anonymized-logs"
              :label="$t('general_settings.labels.anonymized_logs')"
              color="primary"
              :success-messages="settingsMessages['anonymizedLogs'].success"
              :error-messages="settingsMessages['anonymizedLogs'].error"
              @change="onAnonymizedLogsChange($event)"
            />
            <v-switch
              v-model="anonymousUsageAnalytics"
              class="general-settings__fields__anonymous-usage-statistics"
              color="primary"
              :label="$t('general_settings.labels.anonymous_analytics')"
              :success-messages="
                settingsMessages['anonymousUsageAnalytics'].success
              "
              :error-messages="
                settingsMessages['anonymousUsageAnalytics'].error
              "
              @change="onAnonymousUsageAnalyticsChange($event)"
            />
            <v-menu
              ref="historicDateMenu"
              v-model="historicDateMenu"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              max-width="290px"
              min-width="290px"
            >
              <template #activator="{ on }">
                <v-text-field
                  v-model="historicDataStart"
                  class="general-settings__fields__historic-data-start"
                  :label="$t('general_settings.labels.historical_data_start')"
                  :hint="$t('general_settings.historic_start_hint')"
                  prepend-icon="fa-calendar"
                  :success-messages="
                    settingsMessages['historicDataStart'].success
                  "
                  :error-messages="settingsMessages['historicDataStart'].error"
                  readonly
                  v-on="on"
                  @change="onHistoricDataStartChange($event)"
                />
              </template>
              <v-date-picker
                v-model="date"
                no-title
                @input="historicDateMenu = false"
              />
            </v-menu>

            <v-text-field
              v-model="rpcEndpoint"
              class="general-settings__fields__rpc-endpoint"
              :label="$t('general_settings.labels.rpc_endpoint')"
              type="text"
              data-vv-name="eth_rpc_endpoint"
              :success-messages="settingsMessages['rpcEndpoint'].success"
              :error-messages="settingsMessages['rpcEndpoint'].error"
              clearable
              @paste="onRpcEndpointChange($event.clipboardData.getData('text'))"
              @click:clear="onRpcEndpointChange('')"
              @change="onRpcEndpointChange($event)"
            />

            <v-text-field
              v-model="balanceSaveFrequency"
              class="general-settings__fields__balance-save-frequency"
              :label="$t('general_settings.labels.balance_saving_frequency')"
              type="number"
              :success-messages="
                settingsMessages['balanceSaveFrequency'].success
              "
              :error-messages="settingsMessages['balanceSaveFrequency'].error"
              @change="onBalanceSaveFrequencyChange($event)"
            />

            <v-text-field
              v-model="dateDisplayFormat"
              class="general-settings__fields__date-display-format"
              :label="$t('general_settings.labels.date_display_format')"
              type="text"
              :success-messages="settingsMessages['dateDisplayFormat'].success"
              :error-messages="settingsMessages['dateDisplayFormat'].error"
              @change="onDateDisplayFormatChange($event)"
            />
          </v-card-text>
        </v-card>
        <v-card class="mt-5">
          <v-card-title>
            {{ $t('general_settings.amount.title') }}
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="floatingPrecision"
              class="general-settings__fields__floating-precision"
              :label="$t('general_settings.amount.labels.floating_precision')"
              type="number"
              :success-messages="settingsMessages['floatingPrecision'].success"
              :error-messages="settingsMessages['floatingPrecision'].error"
              @change="onFloatingPrecisionChange($event)"
            />

            <v-select
              v-model="selectedCurrency"
              class="general-settings__fields__currency-selector"
              :label="$t('general_settings.amount.labels.main_currency')"
              item-text="ticker_symbol"
              return-object
              :items="currencies"
              :success-messages="settingsMessages['selectedCurrency'].success"
              :error-messages="settingsMessages['selectedCurrency'].error"
              @change="onSelectedCurrencyChange($event)"
            >
              <template #item="{ item, attrs, on }">
                <v-list-item
                  :id="`currency__${item.ticker_symbol.toLocaleLowerCase()}`"
                  v-bind="attrs"
                  v-on="on"
                >
                  <v-list-item-avatar
                    class="general-settings__currency-list primary--text"
                  >
                    {{ item.unicode_symbol }}
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>
                      {{ item.name }}
                    </v-list-item-title>
                    <v-list-item-subtitle>
                      {{
                        $t(
                          'general_settings.amount.labels.main_currency_subtitle'
                        )
                      }}
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </template>
            </v-select>

            <v-text-field
              v-model="thousandSeparator"
              class="general-settings__fields__thousand-separator"
              :label="$t('general_settings.amount.label.thousand_separator')"
              type="text"
              :success-messages="settingsMessages['thousandSeparator'].success"
              :error-messages="settingsMessages['thousandSeparator'].error"
              @change="onThousandSeparatorChange($event)"
            />

            <v-text-field
              v-model="decimalSeparator"
              class="general-settings__fields__decimal-separator"
              :label="$t('general_settings.amount.label.decimal_separator')"
              type="text"
              :success-messages="settingsMessages['decimalSeparator'].success"
              :error-messages="settingsMessages['decimalSeparator'].error"
              @change="onDecimalSeparatorChange($event)"
            />

            <v-radio-group
              v-model="currencyLocation"
              class="general-settings__fields__currency-location"
              :label="$t('general_settings.amount.label.currency_location')"
              row
              :success-messages="settingsMessages['currencyLocation'].success"
              :error-messages="settingsMessages['currencyLocation'].error"
              @change="onCurrencyLocationChange($event)"
            >
              <v-radio
                :label="$t('general_settings.amount.label.location_before')"
                value="before"
              />
              <v-radio
                :label="$t('general_settings.amount.label.location_after')"
                value="after"
              />
            </v-radio-group>

            <strong>
              {{ $t('general_settings.amount.label.resulting_format') }}
            </strong>
            <amount-display :value="amountExample" show-currency="symbol" />
          </v-card-text>
        </v-card>
        <v-card class="mt-5">
          <v-card-title>
            {{ $t('general_settings.frontend.title') }}
          </v-card-title>
          <v-card-text>
            <v-switch
              v-model="scrambleData"
              class="general-settings__fields__scramble-data"
              :label="$t('general_settings.frontend.label.scramble')"
              :success-messages="settingsMessages['scrambleData'].success"
              :error-messages="settingsMessages['scrambleData'].error"
              @change="onScrambleDataChange($event)"
            />
            <time-frame-settings
              :message="settingsMessages['timeframe']"
              :value="defaultGraphTimeframe"
              @timeframe-change="onTimeframeChange"
            />
            <v-text-field
              v-model="queryPeriod"
              class="general-settings__fields__periodic-client-query-period"
              :label="$t('general_settings.frontend.label.query_period')"
              :hint="$t('general_settings.frontend.label.query_period_hint')"
              type="number"
              min="5"
              max="3600"
              :success-messages="settingsMessages['queryPeriod'].success"
              :error-messages="settingsMessages['queryPeriod'].error"
              @change="onQueryPeriodChange($event)"
            />
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Watch } from 'vue-property-decorator';
import { mapActions, mapGetters, mapState } from 'vuex';
import AmountDisplay from '@/components/display/AmountDisplay.vue';
import TimeFrameSettings from '@/components/settings/general/TimeFrameSettings.vue';
import { currencies } from '@/data/currencies';
import { Currency } from '@/model/currency';
import { monitor } from '@/services/monitoring';
import {
  TIMEFRAME_SETTING,
  TIMEFRAME_ALL,
  QUERY_PERIOD
} from '@/store/settings/consts';
import {
  FrontendSettingsPayload,
  TimeFrameSetting
} from '@/store/settings/types';
import { ActionStatus } from '@/store/types';
import {
  CURRENCY_AFTER,
  CurrencyLocation,
  GeneralSettings,
  SettingsUpdate
} from '@/typing/types';
import { bigNumberify } from '@/utils/bignumbers';
import Settings, { SettingsMessages } from '@/views/settings/Settings.vue';

type BaseMessage = { success: string; error: string };

@Component({
  components: {
    TimeFrameSettings,
    AmountDisplay
  },
  computed: {
    ...mapState('session', ['generalSettings']),
    ...mapGetters('session', ['currency'])
  },
  methods: {
    ...mapActions('session', ['settingsUpdate']),
    ...mapActions('settings', ['updateSetting'])
  }
})
export default class General extends Settings {
  generalSettings!: GeneralSettings;
  currency!: Currency;

  floatingPrecision: string = '0';
  anonymizedLogs: boolean = false;
  anonymousUsageAnalytics: boolean = false;
  historicDataStart: string = '';
  rpcEndpoint: string = 'http://localhost:8545';
  balanceSaveFrequency: string = '0';
  dateDisplayFormat: string = '';
  thousandSeparator: string = '';
  decimalSeparator: string = '';
  currencyLocation: CurrencyLocation = CURRENCY_AFTER;
  selectedCurrency: Currency = currencies[0];
  scrambleData: boolean = false;
  defaultGraphTimeframe: TimeFrameSetting = TIMEFRAME_ALL;
  settingsUpdate!: (update: SettingsUpdate) => Promise<ActionStatus>;
  updateSetting!: (payload: FrontendSettingsPayload) => Promise<ActionStatus>;
  queryPeriod: string = '5';

  settingsMessages: SettingsMessages = {
    floatingPrecision: { success: '', error: '' },
    anonymizedLogs: { success: '', error: '' },
    anonymousUsageAnalytics: { success: '', error: '' },
    historicDataStart: { success: '', error: '' },
    rpcEndpoint: { success: '', error: '' },
    balanceSaveFrequency: { success: '', error: '' },
    dateDisplayFormat: { success: '', error: '' },
    thousandSeparator: { success: '', error: '' },
    decimalSeparator: { success: '', error: '' },
    currencyLocation: { success: '', error: '' },
    selectedCurrency: { success: '', error: '' },
    scrambleData: { success: '', error: '' },
    timeframe: { success: '', error: '' },
    queryPeriod: { success: '', error: '' }
  };

  historicDateMenu: boolean = false;
  date: string = '';
  amountExample = bigNumberify(123456.789);

  @Watch('date')
  dateWatch() {
    const date = this.formatDate(this.date);
    if (this.historicDataStart !== date) {
      this.historicDataStart = date;
      this.onHistoricDataStartChange(this.historicDataStart);
    }
  }

  async onTimeframeChange(timeframe: TimeFrameSetting) {
    const { success, message } = await this.updateSetting({
      [TIMEFRAME_SETTING]: timeframe
    });
    this.validateSettingChange(
      'timeframe',
      success ? 'success' : 'error',
      success
        ? `${this.$t('general_settings.validation.timeframe.success', {
            timeframe: timeframe
          })}`
        : `${this.$t('general_settings.validation.timeframe.error', {
            message
          })}`
    );

    if (success) {
      this.defaultGraphTimeframe = timeframe;
    }
  }

  async update(
    update: SettingsUpdate,
    setting: string,
    baseMessage: BaseMessage
  ): Promise<boolean> {
    const { message, success } = await this.settingsUpdate(update);

    this.validateSettingChange(
      setting,
      success ? 'success' : 'error',
      success ? baseMessage.success : `${baseMessage.error}: ${message}`
    );
    return success;
  }

  onScrambleDataChange(enabled: boolean) {
    const { commit } = this.$store;
    const previousValue = this.$store.state.session.scrambleData;

    let success: boolean = false;
    let message: string | undefined;

    try {
      commit('session/scrambleData', enabled);
      success = true;
    } catch (error) {
      this.scrambleData = previousValue;
      message = error.message;
    }

    this.validateSettingChange(
      'scrambleData',
      success ? 'success' : 'error',
      success
        ? ''
        : `${this.$t('general_settings.validation.scramble.error', {
            message
          })}`
    );
  }

  async onQueryPeriodChange(queryPeriod: string) {
    const period = parseInt(queryPeriod);
    if (period < 5 || period > 3600) {
      const message = `${this.$t(
        'general_settings.validation.periodic_query.invalid_period',
        {
          start: 5,
          end: 3600
        }
      )}`;
      this.validateSettingChange(
        'queryPeriod',
        'error',
        `${this.$t('general_settings.validation.periodic_query.error', {
          message
        })}`
      );
      this.queryPeriod = this.$store.state.settings![QUERY_PERIOD].toString();
      return;
    }
    const { success, message } = await this.updateSetting({
      [QUERY_PERIOD]: period
    });
    this.validateSettingChange(
      'queryPeriod',
      success ? 'success' : 'error',
      success
        ? `${this.$t('general_settings.validation.periodic_query.success', {
            period
          })}`
        : `${this.$t('general_settings.validation.periodic_query.error', {
            message
          })}`
    );

    if (success) {
      monitor.stop();
      monitor.start();
    }
  }

  async onSelectedCurrencyChange(currency: Currency) {
    const symbol = currency.ticker_symbol;
    const message: BaseMessage = {
      success: `${this.$t('general_settings.validation.currency.success', {
        symbol
      })}`,
      error: `${this.$t('general_settings.validation.currency.error')}`
    };
    await this.update({ main_currency: symbol }, 'selectedCurrency', message);
  }

  async onThousandSeparatorChange(thousandSeparator: string) {
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.thousand_separator.success',
        { thousandSeparator }
      )}`,
      error: `${this.$t(
        'general_settings.validation.thousand_separator.error'
      )}`
    };

    await this.update(
      { thousand_separator: thousandSeparator },
      'thousandSeparator',
      message
    );
  }

  async onDecimalSeparatorChange(decimalSeparator: string) {
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.decimal_separator.success',
        {
          decimalSeparator
        }
      )}`,
      error: `${this.$t('general_settings.validation.decimal_separator.error')}`
    };

    await this.update(
      { decimal_separator: decimalSeparator },
      'decimalSeparator',
      message
    );
  }

  async onCurrencyLocationChange(currencyLocation: CurrencyLocation) {
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.currency_location.success',
        {
          currencyLocation
        }
      )}`,
      error: `${this.$t('general_settings.validation.currency_location.error')}`
    };

    await this.update(
      { currency_location: currencyLocation },
      'currencyLocation',
      message
    );
  }

  async onFloatingPrecisionChange(precision: string) {
    const previousValue = this.generalSettings.floatingPrecision.toString();

    if (!this.notTheSame(precision, previousValue)) {
      return;
    }

    const params = {
      precision
    };
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.floating_precision.success',
        params
      )}`,
      error: `${this.$t(
        'general_settings.validation.floating_precision.error',
        params
      )}`
    };

    const success = await this.update(
      { ui_floating_precision: parseInt(precision) },
      'floatingPrecision',
      message
    );

    if (!success) {
      this.floatingPrecision = previousValue;
    }
  }

  async onAnonymizedLogsChange(enabled: boolean) {
    const message: BaseMessage = {
      success: '',
      error: `${this.$t('general_settings.validation.anonymized_logs.error')}`
    };

    await this.update({ anonymized_logs: enabled }, 'anonymizedLogs', message);
  }

  async onAnonymousUsageAnalyticsChange(enabled: boolean) {
    const message: BaseMessage = {
      success: '',
      error: `${this.$t('general_settings.validation.analytics.error')}`
    };

    await this.update(
      { submit_usage_analytics: enabled },
      'anonymousUsageAnalytics',
      message
    );
  }

  async onHistoricDataStartChange(date: string) {
    const previousValue = this.parseDate(
      this.generalSettings.historicDataStart
    );

    if (!this.notTheSame(date, previousValue)) {
      return;
    }

    const params = {
      date
    };
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.historic_data.success',
        params
      )}`,
      error: `${this.$t(
        'general_settings.validation.historic_data.error',
        params
      )}`
    };

    const success = await this.update(
      { historical_data_start: date },
      'historicDataStart',
      message
    );

    if (!success) {
      this.historicDataStart = previousValue || '';
    }
  }

  async onBalanceSaveFrequencyChange(frequency: string) {
    const previousValue = this.generalSettings.balanceSaveFrequency;

    if (!this.notTheSame(frequency, previousValue.toString())) {
      return;
    }

    const params = {
      frequency
    };
    const message: BaseMessage = {
      success: `${this.$t(
        'general_settings.validation.balance_frequency.success',
        params
      )}`,
      error: `${this.$t(
        'general_settings.validation.balance_frequency.error',
        params
      )}`
    };

    const success = await this.update(
      { balance_save_frequency: parseInt(frequency) },
      'balanceSaveFrequency',
      message
    );

    if (!success) {
      this.balanceSaveFrequency = previousValue.toString();
    }
  }

  async onDateDisplayFormatChange(dateFormat: string) {
    const message: BaseMessage = {
      success: `${this.$t('general_settings.validation.date_format.success', {
        dateFormat
      })}`,
      error: `${this.$t('general_settings.validation.date_format.error')}`
    };

    await this.update(
      { date_display_format: dateFormat },
      'dateDisplayFormat',
      message
    );
  }

  async onRpcEndpointChange(endpoint: string) {
    const previousValue = this.generalSettings.ethRpcEndpoint;

    if (!this.notTheSame(endpoint, previousValue) && endpoint !== '') {
      return;
    }

    const message: BaseMessage = {
      success: endpoint
        ? `${this.$t('general_settings.validation.rpc.success_set', {
            endpoint
          })}`
        : `${this.$t('general_settings.validation.rpc.success_unset')}`,
      error: `${this.$t('general_settings.validation.rpc.error')}`
    };

    const success = await this.update(
      { eth_rpc_endpoint: endpoint },
      'rpcEndpoint',
      message
    );

    if (!success) {
      this.rpcEndpoint = previousValue || '';
    }
  }

  formatDate(date: string) {
    if (!date) return '';

    const [year, month, day] = date.split('-');
    return `${day}/${month}/${year}`;
  }

  parseDate(date: string) {
    if (
      !/^([0-2]\d|[3][0-1])\/([0]\d|[1][0-2])\/([2][01]|[1][6-9])\d{2}$/.test(
        date
      )
    )
      return null;

    const [day, month, year] = date.split('/');
    return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`;
  }

  get currencies(): Currency[] {
    return currencies;
  }

  mounted() {
    this.loadFromState();
  }

  private loadFromState() {
    this.selectedCurrency = this.currency;
    const settings = this.generalSettings;
    this.floatingPrecision = settings.floatingPrecision.toString();
    this.anonymizedLogs = settings.anonymizedLogs;
    this.anonymousUsageAnalytics = settings.anonymousUsageAnalytics;
    this.historicDataStart = settings.historicDataStart;
    this.rpcEndpoint = settings.ethRpcEndpoint;
    this.balanceSaveFrequency = settings.balanceSaveFrequency.toString();
    this.dateDisplayFormat = settings.dateDisplayFormat;
    this.thousandSeparator = settings.thousandSeparator;
    this.decimalSeparator = settings.decimalSeparator;
    this.currencyLocation = settings.currencyLocation;
    this.date = this.parseDate(settings.historicDataStart) || '';
    const state = this.$store.state;
    this.scrambleData = state.session.scrambleData;
    this.defaultGraphTimeframe = state.settings![TIMEFRAME_SETTING];
    this.queryPeriod = state.settings![QUERY_PERIOD].toString();
  }

  notTheSame<T>(value: T, oldValue: T): T | undefined {
    return value !== oldValue ? value : undefined;
  }
}
</script>
<style scoped lang="scss">
.general-settings {
  &__currency-list {
    font-size: 2em;
    font-weight: bold;
  }

  &__timeframe {
    min-height: 55px;
  }
}
</style>
