<template>
  <main>
    <div style="margin-top: 30px">
      <p class="amount-p">Section One</p>
      <div class="flex" style="margin-top: 10px">
        <div>
          <p class="amount-p">Amount</p>
          <div class="card flex">
            <div>
              <input type="text" class="card-input" v-model="amount" />
            </div>
            <div>
              <AppDropdown
                :list="localFilterList"
                :selectedValue="selectedValueFrom"
                type="from"
                @updateShortCode="convertShortCode"
                @search="searchCurrencies"
              >
                <div class="option" @click="select">id: {{ id }}, title: {{ title }}</div>
              </AppDropdown>
            </div>
          </div>
        </div>
        <div class="card-icon">
          <AppTwoWayIcon />
        </div>
        <div>
          <p class="amount-p">Converted to</p>
          <div class="card flex">
            <div>
              <input type="text" class="card-input" v-model="convertItem.value" disabled />
            </div>
            <div>
              <AppDropdown
                :list="localFilterList"
                type="to"
                :selectedValue="selectedValueTo"
                @updateShortCode="convertShortCode"
              >
                <div class="option" @click="select">id: {{ id }}, title: {{ title }}</div>
              </AppDropdown>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div style="margin-top: 10px">
      <p class="amount-p">Section Two</p>
      <div style="border-radius: 5px; margin-top: 17px; max-width: 300px">
        <AppDropdown
          :list="localFilterList"
          :selectedValue="selectedOnlineValue"
          @updateShortCode="getOnlineReateCode"
          @search="searchCurrencies"
        >
          <div class="option" @click="select">id: {{ id }}, title: {{ title }}</div>
        </AppDropdown>
      </div>
      <div class="flex section-two" style="margin-top: 10px">
        <div class="section-two-left">
          <p class="section-two-p">Currency</p>
        </div>
        <div class="section-two-right">
          <p class="section-two-p">Online Sell Rate</p>
        </div>
      </div>
      <div class="section-two-list" v-for="item in onlineRateList" :key="item.id">
        <template v-if="getContryName(item.id)">
          <div class="flex">
            <div class="section-two-left">
              <p class="section-two-list-p">{{ getContryName(item.id) }} ( {{ item.id }})</p>
            </div>
            <div class="section-two-right">
              <p class="section-two-list-p">{{ item.value }}</p>
            </div>
          </div>
          <div class="section-divider"></div>
        </template>
      </div>
    </div>
  </main>
</template>
<script setup>
import AppDropdown from '../components/AppDropdown.vue'
import AppTwoWayIcon from '../components/icons/AppTwoWayIcon.vue'
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
const pageLoading = ref(true)
const searchValue = ref('')
const countriesList = ref([])
const selectedValueFrom = ref('')
const selectedOnlineValue = ref('')
const selectedValueTo = ref('')
const baseURL = ref('https://api.currencybeacon.com/v1')
const fromShortCode = ref('USD')
const toShortCode = ref('AUD')
const amount = ref('10000')
const convertItem = ref({})
const onlineRateList = ref([])
const cloneOnlineRateList = ref([])

const localFilterList = computed(() => {
  return countriesList.value.filter(
    (list) =>
      list.short_code && list.short_code.toLowerCase().includes(searchValue.value.toLowerCase()),
  )
})

onMounted(async () => {
  pageLoading.value = true
  selectedValueFrom.value = fromShortCode.value
  selectedValueTo.value = toShortCode.value
  await getDefaultCurrency()
  await localIndex()
  await loadLatestOnlineRate()
  pageLoading.value = false
})

const getDefaultCurrency = async () => {
  await axios
    .get(baseURL.value + '/convert', {
      params: {
        api_key: '1SMxQumaAcC996SUXGsnlBPW44t2RR82',
        from: fromShortCode.value,
        to: toShortCode.value,
        amount: amount.value,
      },
    })
    .then((response) => {
      convertItem.value = { ...response.data }
    })
}

const convertShortCode = async (type, code) => {
  if (type === 'from') fromCode(code)
  if (type === 'to') toCode(code)
  await getDefaultCurrency()
}

const getOnlineReateCode = async (type = null, code) => {
  selectedOnlineValue.value = code
  const list = cloneOnlineRateList.value.filter((list) => list.id === code)
  onlineRateList.value = list
}

const fromCode = async (code) => {
  selectedValueFrom.value = code
  fromShortCode.value = code
}

const toCode = async (code) => {
  selectedValueTo.value = code
  toShortCode.value = code
}

const searchCurrencies = async (value) => {
  searchValue.value = value
}

const localIndex = async () => {
  await axios
    .get(baseURL.value + '/currencies', {
      params: { api_key: '1SMxQumaAcC996SUXGsnlBPW44t2RR82' },
    })
    .then((response) => {
      dataCovert(response.data)
    })
}

const loadLatestOnlineRate = async () => {
  await axios
    .get(baseURL.value + '/latest', {
      params: { api_key: '1SMxQumaAcC996SUXGsnlBPW44t2RR82' },
    })
    .then((response) => {
      onlineRateCovert(response.data.rates)
    })
}

const dataCovert = async (data) => {
  const list = []
  for (const [key, value] of Object.entries(data)) {
    list.push(data[key])
  }
  countriesList.value = list
}

const onlineRateCovert = async (data) => {
  const list = []
  for (const [key, value] of Object.entries(data)) {
    list.push({ id: key, value: value })
  }
  onlineRateList.value = list
  cloneOnlineRateList.value = list
}

const getContryName = (code) => {
  const item = countriesList.value.find((list) => list.short_code === code)
  return item ? item.name : null
}
</script>

<style scoped>
.amount-p {
  color: #454745;
  line-height: 20px;
}
.card-section {
  display: flex;
}
.flex {
  display: flex;
}
.card {
  border: 1px solid #c9cbce !important;
  border-radius: 10px;
  margin-top: 10px;
}
.card-input {
  border: none !important;
  border-radius: 10px;
  padding: 18px;
}
.card-icon {
  margin: 45px 10px;
}
input:focus {
  outline: none;
}
.section-two {
  background: #ecefeb;
  width: 100%;
  padding: 20px;
}
.section-two-p {
  color: #6a6a69;
  font-weight: bold;
}
.section-two-left {
  width: 50%;
  float: left;
}
.section-two-right {
  width: 50%;
  float: left;
}
.section-two-list {
  padding: 10px;
}
.section-two-list-p {
  color: #454745;
  line-height: 20px;
}
.section-divider {
  background: #c9cbce;
  width: 100%;
  display: block;
  height: 1px;
  margin: 10px 0px;
}
</style>
