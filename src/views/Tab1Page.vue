<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Incomes</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-refresher slot="fixed" @ionRefresh="doRefresh($event)" class="z-10">
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>

      <div v-if="isLoading" class="absolute flex items-center justify-center inset-0 bg-black opacity-50 z-10">
        <svg class="animate-spin h-5 w-5 mr-3 text-white" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        <p class="text-white text-xl">Loading...</p>
      </div>

      <div class="px-4">
        <div class="flex justify-between items-baseline">
          <Listbox as="div" class="w-1/2" v-model="selected">
            <ListboxLabel class="block text-sm font-medium text-gray-700 sr-only"> Assigned to </ListboxLabel>
            <div class="mt-1 relative">
              <ListboxButton class="relative w-full bg-white font-bold text-lg rounded-md pr-10 py-2 text-left cursor-default focus:outline-none focus:ring-0 sm:text-sm">
                <span class="block truncate">{{ selected.name }}</span>
                <span class="absolute inset-y-0 right-0 flex items-center pr-2 pointer-events-none">
                  <SelectorIcon class="h-5 w-5 text-gray-400" aria-hidden="true" />
                </span>
              </ListboxButton>

              <transition leave-active-class="transition ease-in duration-100" leave-from-class="opacity-100" leave-to-class="opacity-0">
                <ListboxOptions class="absolute z-10 mt-1 w-full bg-white shadow-lg max-h-60 rounded-md py-1 text-base ring-1 ring-black ring-opacity-5 overflow-auto focus:outline-none sm:text-sm">
                  <ListboxOption as="template" v-for="option in options" :key="option.value" :value="option" v-slot="{ active, selected }">
                    <li :class="[active ? 'text-white bg-blue-600' : 'text-gray-900', 'cursor-default select-none relative py-2 pl-8 pr-4']">
                      <span :class="[selected ? 'font-semibold' : 'font-normal', 'block truncate']">
                        {{ option.name }}
                      </span>

                      <span v-if="selected" :class="[active ? 'text-white' : 'text-blue-600', 'absolute inset-y-0 left-0 flex items-center pl-1.5']">
                        <CheckIcon class="h-5 w-5" aria-hidden="true" />
                      </span>
                    </li>
                  </ListboxOption>
                </ListboxOptions>
              </transition>
            </div>
          </Listbox>
          <a @click.prevent="subscribe">Subscribe</a>
        </div>
        <div class="flex justify-between border rounded-md p-2">
          <div>
            <p class="text-sm text-gray-500">Gross Volume</p>
            <p class="text-lg text-gray-900 font-bold">{{results.gross_volume}}</p>
          </div>
          <div>
            <p class="text-sm text-gray-500">Payments</p>
            <p class="text-lg text-gray-900 font-bold">{{results.payment_count}}</p>
          </div>
          <div>
            <p class="text-sm text-gray-500">Customers</p>
            <p class="text-lg text-gray-900 font-bold">{{results.customer_count}}</p>
          </div>
        </div>

        <div class="mt-6">
          <div class="border rounded-md p-2">
            <p class="text-sm text-gray-500">Monthly Recurring Revenue</p>
            <div class="flex justify-between">
              <p class="text-xl text-gray-900 font-bold"><small class="text-xs text-gray-400">curr</small> {{results.mrr.current}}</p>
              <p class="text-xl text-gray-900 font-bold"><small class="text-xs text-gray-400">proj</small> {{results.mrr.projected}}</p>
            </div>
          </div>
        </div>

        <div class="mt-6">
          <div class="border rounded-md p-2">
            <p class="text-sm text-gray-500">Annual Recurring Revenue</p>
            <div class="flex justify-between">
              <p class="text-xl text-gray-900 font-bold"><small class="text-xs text-gray-400">curr</small> {{results.arr.current}}</p>
              <p class="text-xl text-gray-900 font-bold"><small class="text-xs text-gray-400">proj</small> {{results.arr.projected}}</p>
            </div>
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { defineComponent } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonRefresher, IonRefresherContent } from '@ionic/vue';
import {FCM} from "@capacitor-community/fcm";
import apiClient from "@/service/api";
import { ref } from 'vue'
import { Listbox, ListboxButton, ListboxLabel, ListboxOption, ListboxOptions } from '@headlessui/vue'
import { CheckIcon, SelectorIcon } from '@heroicons/vue/solid'

const options = [
  { value: 'today', name: 'Today' },
  { value: 'yesterday', name: 'Yesterday' },
]

export default  defineComponent({
  name: 'Tab1Page',
  components: {
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonRefresher,
    IonRefresherContent,
    Listbox,
    ListboxButton,
    ListboxLabel,
    ListboxOption,
    ListboxOptions,
    CheckIcon,
    SelectorIcon,
  },
  setup() {
    const selected = ref(options[0])

    return {
      options,
      selected,
    }
  },
  data () {
    return {
      isLoading: false,
      results: {
        gross_volumne: 0,
        payment_count: 0,
        customer_count: 0,
        mrr: {
          current: 0,
          projected: 0
        },
        arr: {
          current: 0,
          projected: 0
        }
      }
    }
  },
  mounted () {
    this.getStats()
  },
  watch: {
    selected () {
      this.getStats()
    }
  },
  methods: {
    getStats () {
      this.isLoading = true;
      apiClient.get(`/daily-stats?selected=${this.selected.value}`).then((response) => {
        this.results = response.data
        this.isLoading = false;
      }).catch(() => {
        this.isLoading = false;
      })
    },

    doRefresh (event) {
      this.getStats()

      setTimeout(() => {
        event.target.complete();
      }, 1000);
    },

    subscribe () {
      FCM.subscribeTo({ topic: "sales" })
          .then(() => alert(`subscribed to topic`))
          .catch((err) => console.log(err));
    }
  }
});
</script>
