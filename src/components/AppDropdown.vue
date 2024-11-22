<template>
  <div class="select">
    <div @click="visible = !visible" class="cursor">
      <div class="flex">
        <img :src="getFlag(selectedValue)" class="selected-flag" v-if="selectedValue" />
        <p class="text">{{ selectedValue ? selectedValue : 'All Contries' }}</p>
        <div style="margin: 14px 10px"><AppArrowDown /></div>
      </div>
    </div>
    <div v-if="visible" class="card-options">
      <div>
        <input
          type="text"
          class="card-input"
          placeholder="Search..."
          v-model="searchValue"
          @input="search"
        />
      </div>
      <div class="options">
        <div v-for="item in list" :key="item.id">
          <div class="flex">
            <div class="text cursor flex" @click="select(item.short_code)">
              <img :src="getFlag(item.short_code)" style="width: 17px; margin-right: 9px" />
              {{ item.short_code }}
            </div>
            <div class="selected-icon" v-if="item.short_code === selectedValue">
              <AppTickIcon />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, toRefs } from 'vue'
import AppTickIcon from './icons/AppTickIcon.vue'
import AppArrowDown from './icons/AppArrowDown.vue'

const visible = ref(false)
const searchValue = ref('')

const props = defineProps({
  list: {
    type: Array,
    default: [],
  },
  selectedValue: {
    type: String,
    default: 'AUD',
  },
  type: {
    type: String,
    default: null,
  },
})
const { type, list } = toRefs(props)
const emit = defineEmits(['updateShortCode', 'search'])

const search = async () => {
  emit('search', searchValue.value)
}

const select = async (selectValue) => {
  visible.value = false
  emit('updateShortCode', type.value, selectValue)
}

const getFlag = (code) => {
  const data = code.split('')
  const shortCode = data[0] + data[1]
  return 'https://flagsapi.com/' + shortCode + '/flat/64.png'
}
</script>

<style scoped>
.card-options {
  z-index: 999;
  position: absolute;
  border-radius: 10px;
  box-shadow: inset 0 0 0 1px #e1e1e0;
  padding: 10px;
  background: #fff;
}
.options {
  max-height: 400px;
  max-width: 300px;
  overflow: scroll;
}
.text {
  width: 100%;
  margin: 15px 0px;
  padding: 0px 8px;
  color: #454745;
  line-height: 20px;
}
.cursor {
  cursor: pointer;
}
.flex {
  display: flex;
}
.selected-icon {
  float: right;
  margin: 15px 0px;
  padding: 0px 8px;
}
.card-input {
  width: 100%;
  padding: 6px;
  border-radius: 4px;
  border: 1px solid #e1e1e0;
}
.selected-flag {
  width: 17px;
  margin-right: 0px;
  height: 22px;
  margin-top: 14px;
}
</style>
