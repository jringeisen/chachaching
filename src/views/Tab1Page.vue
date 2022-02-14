<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Incomes</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large"></ion-title>
        </ion-toolbar>
      </ion-header>

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
          <h3>Today</h3>
          <a @click.prevent="subscribe">Subscribe</a>
        </div>
        <div class="flex justify-between border rounded-md p-2">
          <div>
            <p class="text-sm text-gray-500">Gross Volume</p>
            <p class="text-lg text-gray-900 font-bold">{{results.today.gross_volume}}</p>
          </div>
          <div>
            <p class="text-sm text-gray-500">Payments</p>
            <p class="text-lg text-gray-900 font-bold">{{results.today.payment_count}}</p>
          </div>
          <div>
            <p class="text-sm text-gray-500">Customers</p>
            <p class="text-lg text-gray-900 font-bold">{{results.today.customer_count}}</p>
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
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import {FCM} from "@capacitor-community/fcm";
import apiClient from "@/service/api";

export default  defineComponent({
  name: 'Tab1Page',
  components: { IonHeader, IonToolbar, IonTitle, IonContent, IonPage },
  data () {
    return {
      isLoading: false,
      results: {
        today: {
          gross_volumne: 0,
          payment_count: 0,
          customer_count: 0,
        },
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
  methods: {
    getStats () {
      this.isLoading = true;
      apiClient.get('/daily-stats').then((response) => {
        this.results = response.data
        this.isLoading = false;
      }).catch((error) => {
        console.log(error)
        this.isLoading = false;
      })
    },

    subscribe () {
      FCM.subscribeTo({ topic: "sales" })
          .then(() => alert(`subscribed to topic`))
          .catch((err) => console.log(err));
    }
  }
});
</script>
