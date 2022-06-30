<script setup>
import { useVModel } from '@vueuse/core'

import 'flag-icons/css/flag-icons.css'
import { getCountries, getCountryCallingCode, formatNumber, isValidNumber } from 'libphonenumber-js'

import isoCountries from 'i18n-iso-countries'
import { computed, ref, watchEffect } from 'vue'

const countries = getCountries()

const props = defineProps({
  countryCode: {
    required: false,
    type: String,
    default: null
  },

  phoneNumber: {
    required: false,
    type: String,
    default: null
  },

  outlined: {
    required: false,
    type: Boolean,
    default: false
  },

  dense: {
    required: false,
    type: Boolean,
    default: false
  }
})

const emit = defineEmits([
  'update:countryCode',
  'update:phoneNumber'
])

const selectedCountryCode = useVModel(props, 'countryCode', emit)
const phoneNumber = useVModel(props, 'phoneNumber', emit)

const countryCodeToNameMap = ref({})

async function registerCountryLocale (locale = 'en') {
  const localeJson = await import('i18n-iso-countries/langs/en.json')
  isoCountries.registerLocale(localeJson)

  countryCodeToNameMap.value = isoCountries.getNames('en')
}
registerCountryLocale('en')

const filteredCountries = ref([])

const filter = ref('')

const countryCallingCode = computed(() => {
  if (selectedCountryCode.value) {
    return getCountryCallingCode(selectedCountryCode.value)
  }
  return null
})

watchEffect(() => {
  if (filter.value === '') {
    filteredCountries.value = countries
  }

  filteredCountries.value = countries.filter(countryCode => {
    const countryName = countryCodeToNameMap.value[countryCode]
    if (!countryName) { return false }
    return countryName.toLowerCase().includes(filter.value)
  })
})

watchEffect(() => {
  const formatted = formatNumber(phoneNumber.value, selectedCountryCode.value, 'INTERNATIONAL')
  if (formatted) {
    phoneNumber.value = formatted.replace(`+${countryCallingCode.value} `, '')
  }
})

function onPopupHide () {
  filter.value = ''
}

function isValidPhoneNumber (val) {
  return isValidNumber(val, selectedCountryCode.value) || 'Please enter a valid phone number'
}
</script>

<template>
  <div class="row no-wrap">
    <q-select
      v-model="selectedCountryCode"
      :options="filteredCountries"
      input-debounce="0"
      :outlined="props.outlined"
      :dense="props.dense"
      input-class="q-px-md"
      label="Country Code"
      style="min-width: 140px;"
      @popup-hide="onPopupHide"
    >
      <template #before-options>
        <q-input
          v-model="filter"
          square
          dense
          autofocus
          placeholder="Search..."
          input-class="q-px-sm"
          style="position: sticky; top: 0; z-index: 1;"
          bg-color="white"
        />
      </template>
      <template #option="scope">
        <q-item v-bind="scope.itemProps">
          <q-item-section side>
            <span :class="`fi fi-${scope.opt.toLowerCase()}`" />
          </q-item-section>
          <q-item-section>
            <span v-if="countryCodeToNameMap[scope.opt]">
              {{ countryCodeToNameMap[scope.opt] }}
            </span>
          </q-item-section>
        </q-item>
      </template>
      <template #selected-item="scope">
        <q-item-section
          v-if="scope.opt"
          side
        >
          <span :class="`fi fi-${scope.opt.toLowerCase()}`" />
        </q-item-section>
        <q-item-section v-if="scope.opt">
          <span v-if="countryCodeToNameMap[scope.opt]">
            + {{ getCountryCallingCode(scope.opt) }}
          </span>
        </q-item-section>
      </template>
    </q-select>
    <q-input
      v-model="phoneNumber"
      :outlined="props.outlined"
      :dense="props.dense"
      :rules="[isValidPhoneNumber]"
      class="full-width q-pl-xs"
      label="Phone number"
    />
  </div>
</template>
