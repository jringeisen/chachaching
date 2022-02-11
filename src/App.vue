<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script>
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import { defineComponent } from 'vue';
import { FCM } from '@capacitor-community/fcm';
import { PushNotifications } from '@capacitor/push-notifications';

export default defineComponent({
  name: 'App',
  components: {
    IonApp,
    IonRouterOutlet
  },
  data () {
    return {
      fcm: FCM
    }
  },
  async mounted () {
    await PushNotifications.addListener('registration', token => {
      console.info('Registration token: ', token.value);
    });

    await PushNotifications.addListener('registrationError', err => {
      console.error('Registration error: ', err.error);
    });

    await PushNotifications.addListener('pushNotificationReceived', notification => {
      console.log('Push notification received: ', notification);
    });

    await PushNotifications.addListener('pushNotificationActionPerformed', notification => {
      console.log('Push notification action performed', notification.actionId, notification.inputValue);
    });

    let permStatus = await PushNotifications.checkPermissions();

    if (permStatus.receive === 'prompt') {
      permStatus = await PushNotifications.requestPermissions();
    }

    if (permStatus.receive !== 'granted') {
      throw new Error('User denied permissions!');
    }

    await PushNotifications.register();

    const notificationList = await PushNotifications.getDeliveredNotifications();
    console.log('delivered notifications', notificationList);

    const fcmToken = this.fcm.getToken();
    alert(JSON.stringify(fcmToken));
  }
  // mounted () {
  //   console.log('triggered')
  //   // Request permission to use push notifications
  //   // iOS will prompt user and return if they granted permission or not
  //   // Android will just grant without prompting
  //   PushNotifications.requestPermissions().then(result => {
  //     alert("result " + JSON.stringify(result));
  //   });
  //
  //   // Add registration error if there are.
  //   PushNotifications.addListener("registrationError", (error) => {
  //     console.log(`error on register ${JSON.stringify(error)}`);
  //   }),
  //
  //   // Add Notification received
  //   PushNotifications.addListener(
  //       "pushNotificationReceived",
  //       (notification) => {
  //         console.log(`notification ${JSON.stringify(notification)}`);
  //       }
  //   ),
  //
  //   // Add Action performed
  //   PushNotifications.addListener(
  //       "pushNotificationActionPerformed",
  //       async (notification) => {
  //         alert("notification " + notification)
  //         console.log("notification succeeded");
  //       }
  //   ),
  //
  //   // Initialize the registration with FCM Token
  //   PushNotifications.register();
  //   const fcmToken = this.fcm.getToken();
  //   alert(JSON.stringify(fcmToken));
  //   console.log("token:" + JSON.stringify(fcmToken));
  // }
});
</script>
