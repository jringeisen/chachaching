<template>
  <div id="container">
    <a @click.prevent="subscribe">Click to subscribe to sales topic</a>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import { FCM } from '@capacitor-community/fcm';

export default defineComponent({
  name: 'ExploreContainer',
  props: {
    name: String
  },

  methods: {
    subscribe () {
      FCM.subscribeTo({ topic: "sales" })
          .then(() => alert(`subscribed to topic`))
          .catch((err) => console.log(err));
    }
  }
});
</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
