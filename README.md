##About
I built ChaChaChing out of necessity to make my wife happy 😆. After building her online subscription business she wanted to hear a chaching sound everytime a payment came in and she wanted a simple way to view revenue stats, so I built this app.

##What is ChaChaChing?
It's an app that sends push notifications to your phone with a chaching sound effect and also displays revenue stats.

##Getting started
To get started with the ChaChaChing app, there are a few steps you will need to take in order to make it work correctly. I've put together a tutorial [here](https://dev.to/jringeisen/push-notifications-with-laravel-ionic-vue-and-firebase-4g2a) on how to setup Laravel, Ionic-Vue, Capacitor, and Firebase.

Once you have everything setup you will need to create the following files in your root directory:

- `.env.development`
- `.env.production`

The app will need an endpoint from the backend (you'll provide this in the environment file) that includes an object like the following:

```php
{
    'today' => {
        'gross_income' => '$1,000.00',
        'payment_count' => 10,
        'customer_count' => 10
    },
    'mrr' => {
        'current' => '$20,000.00'
        'projected' => '$25,000.00'
    },
    'arr' => {
        'current' => '$100,000.00',
        'projected' => '$120,000.00'
    }
}
```
