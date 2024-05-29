# Vue.js Projenizde Push Bildirimlerini Kullanma

Vue.js projenizde push bildirimlerini kullanmak için Web Push API ve bir push servisi (örneğin Firebase Cloud Messaging, FCM) kullanabilirsiniz. İşte adım adım bir rehber:

## 1. Firebase Projesi Oluşturma
Öncelikle bir Firebase projesi oluşturmalısınız:
1. [Firebase Console](https://console.firebase.google.com/) adresine gidin ve bir proje oluşturun.
2. Projenize girin ve "Cloud Messaging" sekmesine gidin.
3. "Project settings" altındaki "Cloud Messaging" sekmesine gidin ve `Server key` ve `Sender ID` bilgilerinizi alın.

## 2. Vue.js Projenize Firebase SDK Eklemek
Firebase SDK'sını Vue.js projenize eklemek için aşağıdaki adımları izleyin:

1. Firebase SDK'sını yükleyin:
    ```bash
    npm install firebase
    ```

2. Firebase'i projenize dahil edin. Örneğin, `src/main.js` dosyanıza ekleyin:
    ```javascript
    import firebase from 'firebase/app';
    import 'firebase/messaging';

    const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_AUTH_DOMAIN",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_STORAGE_BUCKET",
        messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
        appId: "YOUR_APP_ID",
    };

    firebase.initializeApp(firebaseConfig);

    const messaging = firebase.messaging();

    messaging.onMessage((payload) => {
        console.log('Message received. ', payload);
        // Custom logic to handle the message
    });
    ```

## 3. Service Worker Kurulumu
Push bildirimlerini alabilmek için bir service worker kurmanız gerekiyor.

1. Proje kök dizininde `public` klasörü altında `firebase-messaging-sw.js` adlı bir dosya oluşturun ve aşağıdaki kodu ekleyin:
    ```javascript
    importScripts('https://www.gstatic.com/firebasejs/8.6.1/firebase-app.js');
    importScripts('https://www.gstatic.com/firebasejs/8.6.1/firebase-messaging.js');

    const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_AUTH_DOMAIN",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_STORAGE_BUCKET",
        messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
        appId: "YOUR_APP_ID",
    };

    firebase.initializeApp(firebaseConfig);

    const messaging = firebase.messaging();

    messaging.setBackgroundMessageHandler(function(payload) {
        console.log('[firebase-messaging-sw.js] Received background message ', payload);
        // Customize notification here
        const notificationTitle = 'Background Message Title';
        const notificationOptions = {
            body: 'Background Message body.',
            icon: '/firebase-logo.png'
        };

        return self.registration.showNotification(notificationTitle, notificationOptions);
    });
    ```

## 4. Kullanıcıdan İzin Alma ve Token Alma
Push bildirimleri gönderebilmek için kullanıcıdan izin almanız ve FCM tokenını almanız gerekmektedir.

```javascript
// src/main.js veya uygun bir dosyada

function requestPermission() {
    console.log('Requesting permission...');
    Notification.requestPermission().then((permission) => {
        if (permission === 'granted') {
            console.log('Notification permission granted.');
            getToken();
        } else {
            console.log('Unable to get permission to notify.');
        }
    });
}

function getToken() {
    messaging.getToken({ vapidKey: 'YOUR_VAPID_KEY' }).then((currentToken) => {
        if (currentToken) {
            console.log('Current token:', currentToken);
            // Send the token to your server and save it for later use
        } else {
            console.log('No registration token available. Request permission to generate one.');
        }
    }).catch((err) => {
        console.log('An error occurred while retrieving token. ', err);
    });
}

// Call the requestPermission function when you want to start the push notification process
requestPermission();
