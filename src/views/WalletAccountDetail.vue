<template>
  <div>
    <div v-if="!error">
      <b-card
        bg-variant="secondary"
        style="color: #fff"
      >
        <div
          class="d-flex flex-row align-items-center text-truncate"
        >
          <b-avatar
            id="address-qr"
            rounded
            size="52"
          >
            <feather-icon
              icon="CameraIcon"
              size="32"
            />
          </b-avatar>
          <div class="ml-2">
            <h3
              style="color: #fff"
              class="mb-0"
            >
              Address: <feather-icon
                icon="CopyIcon"
                size="18"
                @click="copy()"
              />
            </h3>
            {{ address }}
          </div>
        </div>
      </b-card>
      <b-card
        class="d-flex flex-row"
      >
        <b-card-header class="pt-0 pl-0 pr-0">
          <b-card-title>Assets</b-card-title>
          <div>
            <b-button
              v-b-modal.operation-modal
              variant="primary"
              size="sm"
              class="mr-25"
              @click="setOperationModalType('Transfer')"
            >
              <feather-icon
                icon="SendIcon"
                class="d-md-none"
              /><small class="d-none d-md-block">Transfer</small>
            </b-button>
            <b-button
              v-b-modal.operation-modal
              variant="danger"
              size="sm"
              @click="setOperationModalType('IBCTransfer')"
            ><feather-icon
               icon="SendIcon"
               class="d-md-none"
             />
              <span class="d-none d-md-block">IBC Transfer</span>
            </b-button>
          </div>
        </b-card-header>
        <b-card-body class="pl-0 pr-0">
          <b-row>
            <b-col
              xm="12"
              md="4"
            >
              <chart-component-doughnut
                v-if="chartData"
                :height="235"
                :width="235"
                :data="chartData"
                class="mb-3"
              />
            </b-col>
            <b-col
              class="border-left d-none d-md-block"
              md="1"
            />
            <b-col
              xm="12"
              md="7"
            >
              <!-- tokens -->
              <div
                v-for="(token, index) in assetTable.items"
                :key="`asset-${index}`"
                class="d-flex justify-content-between mb-1"
              >
                <div class="d-flex align-items-center">
                  <b-avatar
                    :variant="`light-${token.color}`"
                    rounded
                  >
                    <feather-icon
                      :icon="token.icon"
                      size="16"
                      :class="`text-${token.color}`"
                    />
                  </b-avatar>
                  <span class="font-weight-bold ml-75 d-none d-md-block">{{ token.type }} </span>
                  <span class="ml-25">{{ token.percent }}%</span>
                </div>
                <div class="d-flex flex-column">
                  <span
                    v-b-tooltip.hover.top="token.denom"
                    class="text-right"
                  >{{ formatToken(token) }}</span>
                  <small class="text-right">{{ currency }}{{ formatNumber(token.currency) }}</small>
                </div>
              </div>
              <!--/ tokens -->
              <div class="text-right border-top pt-1">
                <h2>Total: {{ currency }}{{ formatNumber(assetTable.currency) }}</h2>
              </div>
            </b-col>
          </b-row>
        </b-card-body>
      </b-card>
      <b-card
        v-if="unbonding && unbonding.length > 0"
      >
        <b-card-header class="pt-0 pl-0 pr-0">
          <b-card-title>Unbonding Tokens</b-card-title>
        </b-card-header>
        <b-card-body class="pl-0 pr-0">
          <b-row
            v-for="item in unbonding"
            :key="item.validator_address"
          >
            <b-col cols="12">
              <span class="font-weight-bolder">From: <router-link :to="`../staking/${item.validator_address}`">{{ item.validator_address }}</router-link></span>
            </b-col>
            <b-col cols="12">
              <b-table
                :items="item.entries"
                class="mt-1"
                striped
                hover
                responsive="sm"
                stacked="sm"
              >
                <template #cell(completion_time)="data">
                  {{ formatDate(data.item.completion_time) }}
                </template>
                <template #cell(initial_balance)="data">
                  {{ data.item.initial_balance }}{{ stakingParameters.bond_denom }}
                </template>
                <template #cell(balance)="data">
                  {{ data.item.balance }}{{ stakingParameters.bond_denom }}
                </template>
              </b-table>
            </b-col>
          </b-row>
        </b-card-body>
      </b-card>
      <b-card
        v-if="delegations"
      >
        <b-card-header class="pt-0 pl-0 pr-0">
          <b-card-title>Delegation</b-card-title>
          <div>
            <b-button
              v-b-modal.operation-modal
              variant="primary"
              size="sm"
              class="mr-25"
              @click="setOperationModalType('Delegate')"
            >
              <feather-icon
                icon="LogInIcon"
                class="d-md-none"
              /><small class="d-none d-md-block">Delegate</small>
            </b-button>
            <b-button
              v-if="delegations"
              v-b-modal.operation-modal
              variant="primary"
              size="sm"
              @click="setOperationModalType('Withdraw')"
            >
              <feather-icon
                icon="ShareIcon"
                class="d-md-none"
              /><small class="d-none d-md-block"> Withdraw Rewards</small>
            </b-button>
          </div>
        </b-card-header>
        <b-card-body class="pl-0 pr-0">
          <b-table
            :items="deleTable"
            stacked="sm"
          >
            <template #cell(action)="data">
              <!-- size -->
              <b-button-group
                size="sm"
              >
                <b-button
                  v-b-modal.operation-modal
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  v-b-tooltip.hover.top="'Delegate'"
                  variant="outline-primary"
                  @click="selectValue(data.value,'Delegate')"
                >
                  <feather-icon icon="LogInIcon" />
                </b-button>
                <b-button
                  v-b-modal.operation-modal
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  v-b-tooltip.hover.top="'Redelegate'"
                  variant="outline-primary"
                  @click="selectValue(data.value,'Redelegate')"
                >
                  <feather-icon icon="ShuffleIcon" />
                </b-button>
                <b-button
                  v-b-modal.operation-modal
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  v-b-tooltip.hover.top="'Unbond'"
                  variant="outline-primary"
                  @click="selectValue(data.value,'Unbond')"
                >
                  <feather-icon icon="LogOutIcon" />
                </b-button>
              </b-button-group>
            </template>
          </b-table>
        </b-card-body>
      </b-card>

      <b-card title="Transactions">
        <b-table
          :items="txs"
          striped
          hover
          responsive="sm"
          stacked="sm"
        >
          <template #cell(height)="data">
            <router-link :to="`../blocks/${data.item.height}`">
              {{ data.item.height }}
            </router-link>
          </template>
          <template #cell(txhash)="data">
            <router-link :to="`../tx/${data.item.txhash}`">
              {{ formatHash(data.item.txhash) }}
            </router-link>
          </template>
        </b-table>

        <b-pagination
          v-if="Number(transactions.page_total) > 1"
          :total-rows="transactions.total_count"
          :per-page="transactions.limit"
          :value="transactions.page_number"
          align="center"
          class="mt-1"
          @change="pageload"
        />
      </b-card>

      <b-card
        v-if="account"
        title="Profile"
        class="text-trancate"
      >
        <b-table-simple stacked="sm">
          <b-tbody v-if="account.type === 'cosmos-sdk/BaseAccount'">
            <b-tr>
              <b-td>
                Account Type
              </b-td><b-td> {{ account.type }} </b-td>
            </b-tr>
            <b-tr>
              <b-td class="max-width:100px;">
                Account Number
              </b-td><b-td> {{ account.value.account_number }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Sequence </b-td><b-td> {{ account.value.sequence }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Public Key </b-td><b-td> <object-field-component :tablefield="account.value.public_key" /> </b-td>
            </b-tr>
          </b-tbody>
          <b-tbody v-else-if="account.type === 'cosmos-sdk/PeriodicVestingAccount' && account.value.base_vesting_account">
            <b-tr>
              <b-td>
                Account Type
              </b-td>
              <b-td>
                {{ account.type }}
              </b-td>
            </b-tr>
            <b-tr>
              <b-td>
                Account Number
              </b-td><b-td> {{ account.value.base_vesting_account.base_account.account_number }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Sequence </b-td><b-td> {{ account.value.base_vesting_account.base_account.sequence }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Public Key </b-td><b-td> <object-field-component :tablefield="account.value.base_vesting_account.base_account.public_key" /> </b-td>
            </b-tr>
            <b-tr>
              <b-td> Original Vesting </b-td><b-td> {{ formatToken(account.value.base_vesting_account.original_vesting) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Delegated Free </b-td><b-td> {{ formatToken(account.value.base_vesting_account.delegated_free) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Delegated Vesting </b-td><b-td> {{ formatToken(account.value.base_vesting_account.delegated_vesting) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Vesting Time </b-td><b-td> {{ formatTime(account.value.start_time) }} - {{ formatTime(account.value.base_vesting_account.end_time) }}</b-td>
            </b-tr>
            <b-tr>
              <b-td> Vesting Periods </b-td>
              <b-td>
                <b-table-simple>
                  <th>Length</th><th>Amount</th>
                  <b-tr
                    v-for="p, index in account.value.vesting_periods"
                    :key="index"
                  >
                    <td><small>{{ p.length }} <br>{{ formatLength(p.length) }}</small> </td><td>{{ formatToken(p.amount) }}</td>
                  </b-tr>
                </b-table-simple>
              </b-td>
            </b-tr>
          </b-tbody>
          <b-tbody v-else-if="account.type === 'cosmos-sdk/DelayedVestingAccount' && account.value.base_vesting_account">
            <b-tr>
              <b-td>
                Account Type
              </b-td><b-td> {{ account.type }} </b-td>
            </b-tr>
            <b-tr>
              <b-td style="max-width:100px;">
                Account Number
              </b-td><b-td> {{ account.value.base_vesting_account.base_account.account_number }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Sequence </b-td><b-td> {{ account.value.base_vesting_account.base_account.sequence }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Public Key </b-td><b-td> <object-field-component :tablefield="account.value.base_vesting_account.base_account.public_key" /> </b-td>
            </b-tr>
            <b-tr>
              <b-td> Original Vesting </b-td><b-td> {{ formatToken(account.value.base_vesting_account.original_vesting) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Delegated Free </b-td><b-td> {{ formatToken(account.value.base_vesting_account.delegated_free) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> Delegated Vesting </b-td><b-td> {{ formatToken(account.value.base_vesting_account.delegated_vesting) }} </b-td>
            </b-tr>
            <b-tr>
              <b-td> End Time </b-td><b-td> {{ formatTime(account.value.base_vesting_account.end_time) }}</b-td>
            </b-tr>
          </b-tbody>
          <object-field-component
            v-else
            :tablefield="account.value || account"
          />
        </b-table-simple>
      </b-card>

      <b-popover
        target="address-qr"
        variant="dark"
        triggers="hover"
        placement="bottom"
      >
        <vue-qr :text="address" />
      </b-popover>

      <operation-modal
        :type="operationModalType"
        :address="address"
        :validator-address="selectedValidator"
      />
      <div id="txevent" />
    </div>
    <div v-else>
      <div class="misc-inner p-2 p-sm-3">
        <div class="w-100 text-center">
          <h2 class="mb-1">
            Account not found  🕵🏻‍♀️
          </h2>
          <p class="mb-2">
            Oops! 😖 {{ error }}.
          </p>

          <b-button
            variant="primary"
            class="mb-2 btn-sm-block"
            :to="{path:'../'}"
          >
            Back to home
          </b-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { $themeColors } from '@themeConfig'
import dayjs from 'dayjs'
import {
  BCard, BAvatar, BPopover, BTable, BRow, BCol, BTableSimple, BTr, BTd, BTbody, BCardHeader, BCardTitle, BButton, BCardBody, VBModal,
  BButtonGroup, VBTooltip, BPagination,
} from 'bootstrap-vue'
import FeatherIcon from '@/@core/components/feather-icon/FeatherIcon.vue'
import ToastificationContent from '@core/components/toastification/ToastificationContent.vue'
import Ripple from 'vue-ripple-directive'
import VueQr from 'vue-qr'
import chainAPI from '@/libs/fetch'
import {
  formatToken, formatTokenAmount, formatTokenDenom, getStakingValidatorOperator, percent, tokenFormatter, toDay,
  toDuration, abbrMessage, abbrAddress, getUserCurrency, getUserCurrencySign, numberWithCommas,
} from '@/libs/utils'
import OperationModal from '@/views/components/OperationModal/index.vue'
import ObjectFieldComponent from './ObjectFieldComponent.vue'
import ChartComponentDoughnut from './ChartComponentDoughnut.vue'

export default {
  components: {
    BRow,
    BCol,
    BCard,
    BAvatar,
    BPopover,
    BTable,
    FeatherIcon,
    VueQr,
    BTableSimple,
    BTbody,
    BCardHeader,
    BCardTitle,
    BCardBody,
    BButton,
    BButtonGroup,
    BTr,
    BTd,
    BPagination,
    // eslint-disable-next-line vue/no-unused-components
    ToastificationContent,
    ObjectFieldComponent,
    ChartComponentDoughnut,
    OperationModal,
  },
  directives: {
    'b-modal': VBModal,
    'b-tooltip': VBTooltip,
    Ripple,
  },
  data() {
    const { address } = this.$route.params
    return {
      currency: getUserCurrencySign(),
      selectedValidator: '',
      totalCurrency: 0,
      address,
      account: null,
      assets: [],
      reward: [],
      delegations: [],
      redelegations: [],
      unbonding: [],
      quotes: {},
      transactions: [],
      stakingParameters: {},
      operationModalType: '',
      error: null,
    }
  },
  computed: {
    accountTitle() {
      if (this.account && this.account.type) {
        return this.account.type.substring(this.account.type.indexOf('/') + 1)
      }
      return 'Profile'
    },
    txs() {
      if (this.transactions.txs) {
        return this.transactions.tx_responses.map(x => ({
          height: Number(x.height),
          txhash: x.txhash,
          msgs: abbrMessage(x.tx.body.messages),
          time: toDay(x.timestamp),
        }))
      }
      return []
    },
    assetTable() {
      let total = []
      let sum = 0
      let sumCurrency = 0
      total = total.concat(this.assets.map(x => {
        const xh = x
        xh.type = 'Balance'
        xh.color = 'success'
        xh.icon = 'CreditCardIcon'
        xh.currency = this.formatCurrency(xh.amount, xh.denom)
        sumCurrency += xh.currency
        sum += Number(xh.amount)
        return xh
      }))

      let stakingDenom = ''

      if (this.delegations && this.delegations.length > 0) {
        let temp = 0
        this.delegations.forEach(x => {
          const xh = x.balance
          temp += Number(xh.amount)
          sumCurrency += this.formatCurrency(xh.amount, xh.denom)
          sum += Number(xh.amount)
          stakingDenom = xh.denom
        })
        total.push({
          type: 'Delegation',
          color: 'primary',
          icon: 'LockIcon',
          amount: temp,
          denom: stakingDenom,
          currency: this.formatCurrency(temp, stakingDenom),
        })
      }

      if (this.reward.total) {
        total = total.concat(this.reward.total.map(x => {
          const xh = x
          xh.type = 'Reward'
          xh.color = 'warning'
          xh.icon = 'TrendingUpIcon'
          xh.currency = this.formatCurrency(xh.amount, xh.denom)
          sumCurrency += xh.currency
          sum += Number(xh.amount)
          return xh
        }))
      }
      if (this.unbonding) {
        let tmp1 = 0
        this.unbonding.forEach(x => {
          x.entries.forEach(e => {
            tmp1 += Number(e.balance)
          })
        })
        if (this.stakingParameters) stakingDenom = this.stakingParameters.bond_denom
        const unbonding = this.formatCurrency(tmp1, stakingDenom)
        sumCurrency += unbonding
        sum += tmp1
        total.push({
          type: 'unbonding',
          color: 'danger',
          icon: 'TrendingDownIcon',
          denom: stakingDenom,
          amount: tmp1,
          percent: 0,
          currency: unbonding,
        })
      }
      total = total.map(x => {
        const xh = x
        xh.percent = percent(Number(x.amount) / sum)
        return xh
      })
      return {
        items: total,
        currency: parseFloat(sumCurrency.toFixed(2)),
      }
    },
    chartData() {
      const data = this.assetTable.items.reduce((t, c) => {
        const th = t
        if (t[c.type]) {
          th[c.type] += Number(c.amount)
        } else {
          th[c.type] = Number(c.amount)
        }
        return th
      }, [])
      return {
        datasets: [
          {
            labels: Object.keys(data),
            data: Object.values(data),
            backgroundColor: [$themeColors.success, $themeColors.primary, $themeColors.warning, $themeColors.danger, $themeColors.info],
            borderWidth: 0,
            pointStyle: 'rectRounded',
          },
        ],
      }
    },
    deleTable() {
      const re = []
      if (this.reward.rewards && this.delegations && this.delegations.length > 0) {
        this.delegations.forEach(e => {
          const reward = this.reward.rewards.find(r => r.validator_address === e.delegation.validator_address)
          re.push({
            validator: getStakingValidatorOperator(this.$http.config.chain_name, e.delegation.validator_address, 8),
            token: formatToken(e.balance, {}, 2),
            reward: tokenFormatter(reward.reward, this.denoms),
            action: e.delegation.validator_address,
          })
        })
      }
      return re
    },
    accTable() {
      let table = {}
      if (this.account && this.account.type === 'cosmos-sdk/PeriodicVestingAccount') {
        table = this.account.value
      }
      return table
    },
    denoms() {
      return this.$store.state.chains.denoms
    },
  },
  created() {
    this.$http.getAuthAccount(this.address).then(acc => {
      this.account = acc
      this.initial()
      this.$http.getTxsBySender(this.address).then(res => {
        this.transactions = res
      })
      this.$http.getStakingParameters().then(res => {
        this.stakingParameters = res
      })
    }).catch(err => {
      this.error = err
    })
  },
  mounted() {
    const elem = document.getElementById('txevent')
    elem.addEventListener('txcompleted', () => {
      this.initial()
    })
  },
  methods: {
    initial() {
      this.$http.getBankAccountBalance(this.address).then(bal => {
        this.assets = bal
        bal.forEach(x => {
          const symbol = formatTokenDenom(x.denom)
          if (!this.quotes[symbol] && symbol.indexOf('/') === -1) {
            chainAPI.fetchTokenQuote(symbol).then(quote => {
              this.$set(this.quotes, symbol, quote)
            })
          }
        })
      })
      this.$http.getStakingReward(this.address).then(res => {
        this.reward = res
      })
      this.$http.getStakingDelegations(this.address).then(res => {
        this.delegations = res.delegation_responses || res
      })
      this.$http.getStakingUnbonding(this.address).then(res => {
        this.unbonding = res.unbonding_responses || res
      })
    },
    formatNumber(v) {
      return numberWithCommas(v)
    },
    pageload(v) {
      this.$http.getTxsBySender(this.address, v).then(res => {
        this.transactions = res
      })
    },
    selectValue(v, type) {
      this.selectedValidator = v
      this.setOperationModalType(type)
    },
    setOperationModalType(type) {
      this.operationModalType = type
    },
    formatHash: abbrAddress,
    formatDenom(v) {
      return formatTokenDenom(this.denoms[v] ? this.denoms[v] : v)
    },
    formatAmount(v, dec = 2, denom = 'uatom', format = true) {
      return formatTokenAmount(v, dec, denom, format)
    },
    formatToken(v) {
      return tokenFormatter(v, this.denoms)
    },
    formatCurrency(amount, denom) {
      const qty = this.formatAmount(amount, 2, denom, false)
      const d2 = this.formatDenom(denom)
      const userCurrency = getUserCurrency()
      const quote = this.$store.state.chains.quotes[d2]
      if (quote) {
        const price = quote[userCurrency]
        return parseFloat((qty * price).toFixed(2))
      }
      return 0
    },
    formatDate: v => dayjs(v).format('YYYY-MM-DD HH:mm:ss'),
    formatTime: v => toDay(Number(v) * 1000),
    formatLength: v => toDuration(Number(v) * 1000),
    copy() {
      this.$copyText(this.address).then(() => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Address copied',
            icon: 'BellIcon',
          },
        })
      }, e => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: `Failed to copy address! ${e}`,
            icon: 'BellIcon',
            variant: 'danger',
          },
        })
      })
    },
  },
}
</script>
