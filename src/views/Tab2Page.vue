<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Posts</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">

      <ion-button @click="escanerQR">Escanear QR</ion-button>
      <ion-note v-if="resultado" color="primary">{{ resultado }}</ion-note>

      <ion-list>
        <ion-item v-for="post in posts" :key="post.id" :router-link="`/tabs/users/${post.userId}`">
          <ion-label>
            <h2>{{ post.title }}</h2>
            <p>{{ post.body }}</p>
          </ion-label>
        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { Posts2 } from '@/interfaces/posts2';
import {
  CapacitorBarcodeScanner,
  CapacitorBarcodeScannerTypeHint,
  CapacitorBarcodeScannerAndroidScanningLibrary,
} from '@capacitor/barcode-scanner';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel, IonButton, IonNote } from '@ionic/vue';
import { ref } from 'vue';

const posts = ref<Posts2[]>([]);
const resultado = ref('');

function cargarPosts() {
  fetch('https://jsonplaceholder.typicode.com/posts')
    .then((response) => response.json())
    .then((json) => {
      console.log(json);
      posts.value = json;
    });
}
cargarPosts();

async function escanerQR() {
  try {
    const data = await CapacitorBarcodeScanner.scanBarcode({
      hint: CapacitorBarcodeScannerTypeHint.QR_CODE,
      scanInstructions: 'Escaneá un código QR',
      android: {
        scanningLibrary: CapacitorBarcodeScannerAndroidScanningLibrary.ZXING,
      },
    });
    console.log(data);
    resultado.value = data.ScanResult;
  } catch (error) {
    // el usuario canceló la cámara o el dispositivo no tiene cámara
    console.log('Escaneo cancelado:', error);
    resultado.value = '';
  }
}
</script>
