<script lang="ts">
import data from '../data/data.json'

export default {
  data() {
    return {
      APY: 0,
      SOL_USD: 0,
      amount: '',
      number: '',
      period: 'day',
      totalSol: '-',
      totalUsd: '-',
      data: {
        perDay: {sol: '-', usd: '-'},
        perMonth: {sol: '-', usd: '-'},
        perYear: {sol: '-', usd: '-'},
      },
      errors: {
        amount: false,
        number: false,
      }
    }
  },
  mounted() {
    this.APY = data.APY;
    this.SOL_USD = data.SOL_USD;
  },
  methods: {
    toReadableMode(value: number): string {
      return isNaN(value) ? value.toString() : (+value).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    addUsdSymbol(value: number | string) {
      return value && value !== '-' ? `$ ${this.toReadableMode(+value)}` : value
    },
    calculateNumberRange(newValue: string): string {
      const periodMap = {
        day: 365,
        month: 12,
        year: 20,
      }
      const minValue = 1;
      const maxValue = periodMap[this.period];
      const value = Math.min(maxValue, Math.max(minValue, +newValue)).toString();
      return value.split('.').slice(0, 2).join('.')
    },
    changeAmountHandler(event: KeyboardEvent) {
      this.errors.amount = false;
      if (isNaN(Number(event.key)) && event.key !== '.') {
        return event.preventDefault();
      }
    },
    changeNumberHandler(event: KeyboardEvent) {
      this.errors.number = false;
      if (isNaN(Number(event.key))) {
        return event.preventDefault();
      }
    },
    blurNumberHandler() {
      this.number = this.calculateNumberRange(this.number);
    },
    changePeriodHandler(event: any) {
      this.period = event.target.value;
      this.number = this.calculateNumberRange(this.number);
    },
    validateFields() {
      this.errors.amount = !this.amount;
      this.errors.number = !this.number;
    },
    calculate() {
      this.validateFields();
      if (this.errors.amount || this.errors.number) {
        return;
      }

      const periodMap = {
        day: this.number / 365,
        month: this.number / 12,
        year: this.number,
      }

      const period = periodMap[this.period];
      this.totalSol = this.amount * ((1 + this.APY / 100) ** period);
      this.totalUsd = (this.totalSol * this.SOL_USD);

      const map = {
        day: this.number,
        month: this.number * 365 / 12,
        year: this.number * 365,
      }
      const perDaySol = (this.totalSol - this.amount) / map[this.period];
      const perMonthSol = perDaySol * 365 / 12;
      const perYearSol = perDaySol * 365;
      this.data = {
        perDay: {sol: perDaySol, usd: perDaySol * this.SOL_USD},
        perMonth: {sol: perMonthSol, usd: perMonthSol * this.SOL_USD},
        perYear: {sol: perYearSol, usd: perYearSol * this.SOL_USD},
      }
    }
  }
}
</script>

<template>
  <div class="calculator">
    <div class="calculator__header">
      <p class="calculator__title">Solana APY Interest Calculator</p>
    </div>
    <div class="calculator__body">
      <div class="calculator__form">
        <div class="rate mb-12">
          <p class="rate__title">Current APY Interest Rate</p>
          <p class="rate__value">≈{{ APY }}%</p>
        </div>

        <div class="control mb-12">
          <label class="control__label">SOL Investment Amount</label>
          <input
              type="number"
              class="control__field"
              :class="{'field-error': errors.amount}"
              v-model="amount"
              @keypress="changeAmountHandler"
          >
        </div>
        <div class="control mb-12">
          <label class="control__label">Investment Tenure</label>
          <div class="d-flex gap-6">
            <input
                type="number"
                class="control__field control__field_number"
                :class="{'field-error': errors.number}"
                v-model="number"
                @keypress="changeNumberHandler"
                @blur='blurNumberHandler'
            >
            <select class="control__field flex-grow-1" :value="period" @change="changePeriodHandler">
              <option value="day">Day</option>
              <option value="month">Month</option>
              <option value="year">Year</option>
            </select>
          </div>
        </div>

        <button class="app-button app-button_primary app-button_block" @click="calculate">
          CALCULATE
        </button>
      </div>
      <div class="calculator__info">
        <table class="app-table app-table_block">
          <tr>
            <th>Total Return SOL</th>
            <th>{{ this.toReadableMode(totalSol) }}</th>
            <th>{{ addUsdSymbol(totalUsd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Day</td>
            <td>{{ this.toReadableMode(data.perDay.sol) }}</td>
            <th>{{ addUsdSymbol(data.perDay.usd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Month</td>
            <td>{{ this.toReadableMode(data.perMonth.sol) }}</td>
            <th>{{ addUsdSymbol(data.perMonth.usd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Year</td>
            <td>{{ this.toReadableMode(data.perYear.sol) }}</td>
            <th>{{ addUsdSymbol(data.perYear.usd) }}</th>
          </tr>
          <tr>
            <td></td>
            <td></td>
            <td></td>
          </tr>
        </table>
        <div>
          <div class="how-to-calculate">
            <p class="how-to-calculate__title">How to calculate APY</p>
            <p class="how-to-calculate__value">Current Price (USD) 1 SOL = {{ SOL_USD }} $</p>
          </div>
          <div>
            <p class="description">
              Annual percentage yield (APY) is the effective annual rate, or real rate, of return of an investment if
              the interest earned each period is compounded. APY considers the effects of compounding, since advertised
              rates are typically the rates of return for simple interest.
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.how-to-calculate {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.how-to-calculate__title {
  font-size: 11px;
  font-weight: 400;
  color: var(--app-black-1);
}

.how-to-calculate__value {
  font-weight: 400;
  font-size: 10px;
  line-height: 12px;
  color: var(--app-gray-2);
}

.description {
  font-size: 10px;
  line-height: 15px;
  font-weight: 300;
  color: var(--app-black-1);
}
</style>
