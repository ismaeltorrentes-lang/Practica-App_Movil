<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Tab 3</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Tab 3</ion-title>
        </ion-toolbar>
      </ion-header>

      <ExploreContainer name="Tab 3 page" />

      <ion-list inset>
        <ion-item>
          <ion-input label="Título" label-placement="floating" v-model="titulo" placeholder="Escribí el título"></ion-input>
        </ion-item>
        <ion-item>
          <ion-textarea label="Body" label-placement="floating" v-model="cuerpo" placeholder="Escribí el contenido" :rows="3"></ion-textarea>
        </ion-item>
        <ion-item>
          <ion-select label="Usuario" label-placement="floating" v-model="userId" placeholder="Elegí un usuario">
            <ion-select-option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.id">
              {{ usuario.name }}
            </ion-select-option>
          </ion-select>
        </ion-item>
      </ion-list>

      <ion-button @click="publicar" :disabled="cargando || !titulo || !cuerpo || !userId">
        {{ cargando ? 'Enviando...' : 'Enviar' }}
      </ion-button>

      <ion-button @click="escanerQR">Escanear QR</ion-button>

      <ion-note v-if="resultadoScan" color="primary">QR escaneado: {{ resultadoScan }}</ion-note>

      <ion-note v-if="error" color="danger">{{ error }}</ion-note>

      <ion-card v-if="respuesta">
        <ion-card-header>
          <ion-card-title>Post creado (id {{ respuesta.id }})</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <p><strong>título:</strong> {{ respuesta.title }}</p>
          <p><strong>body:</strong> {{ respuesta.body }}</p>
          <p><strong>usuario:</strong> {{ nombreUsuario(respuesta.userId) }} (id {{ respuesta.userId }})</p>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">

import {
  
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonNote,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonList,
  IonItem,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption,
} from '@ionic/vue';
import ExploreContainer from '@/components/ExploreContainer.vue';
import { Posts3 } from '@/interfaces/posts3';
import { Usuarios } from '@/interfaces/usuarios';
import { CapacitorBarcodeScanner, CapacitorBarcodeScannerTypeHint } from '@capacitor/barcode-scanner';
import { ref } from 'vue';

const titulo = ref('');
const cuerpo = ref('');
const userId = ref<number | null>(null);

const usuarios = ref<Usuarios[]>([]);
const respuesta = ref<Posts3 | null>(null);
const cargando = ref(false);
const error = ref('');
const resultadoScan = ref('');

function cargarUsuarios() {
  fetch('https://jsonplaceholder.typicode.com/users')
    .then((response) => response.json())
    .then((json) => {
      /**
       * Primero el valor y después las opciones: ion-select solo se vuelve a
       * renderizar cuando cambian sus ion-select-option (MutationObserver),
       * no cuando cambia el value. Si hacemos al revés, queda el placeholder.
       */
      if (json.length > 0) {
        userId.value = json[0].id;
      }
      usuarios.value = json;
    })
    .catch(() => {
      error.value = 'No se pudieron cargar los usuarios';
    });
}
cargarUsuarios();

function nombreUsuario(id: number): string {
  return usuarios.value.find((usuario) => usuario.id === id)?.name ?? '-';
}

function publicar() {
  cargando.value = true;
  error.value = '';

  fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    body: JSON.stringify({
      title: titulo.value,
      body: cuerpo.value,
      userId: userId.value,
    }),
    headers: {
      'Content-type': 'application/json; charset=UTF-8',
    },
  })
    .then((response) => response.json())
    .then((json) => {
      console.log(json);
      respuesta.value = json;
    })
    .catch(() => {
      error.value = 'No se pudo conectar con la API';
    })
    .finally(() => {
      cargando.value = false;
    });
}

async function escanerQR() {
  try {
    const data = await CapacitorBarcodeScanner.scanBarcode({
      hint: CapacitorBarcodeScannerTypeHint.QR_CODE,
      scanInstructions: 'Escaneá un código QR',
    });
    console.log(data);
    resultadoScan.value = data.ScanResult;
  } catch (errorEscaneo) {
    // el usuario canceló la cámara o el dispositivo no tiene cámara
    console.log('Escaneo cancelado:', errorEscaneo);
    resultadoScan.value = '';
  }
}
</script>
