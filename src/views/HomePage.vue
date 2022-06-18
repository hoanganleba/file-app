<template>
  <ion-page>
    <ion-header translucent>
      <ion-toolbar>
        <ion-buttons v-if="$route.params.path" slot="start">
          <ion-back-button @click="$router.go(-1)"></ion-back-button>
        </ion-buttons>
        <ion-title>{{
          $route.params.path
            ? $route.params.path[$route.params.path.length - 1]
            : 'Home'
        }}</ion-title>
      </ion-toolbar>
      <ion-toolbar>
        <ion-searchbar
          placeholder="Search Folder"
          type="text"
          @ionChange="onSearchChange($event)"
        ></ion-searchbar>
      </ion-toolbar>
    </ion-header>
    <ion-content fullscreen class="ion-padding-bottom">
      <ion-list>
        <ion-item
          :lines="isImage(item) ? 'none' : null"
          v-for="(item, index) in items"
          :key="index"
        >
          <ion-label
            class="pl-4"
            v-if="!isImage(item)"
            @click="navigate(item)"
            >{{ item }}</ion-label
          >
          <ion-img
            class="p-2"
            v-if="isImage(item)"
            :src="getImage(item)"
            :alt="item"
          ></ion-img>
        </ion-item>
      </ion-list>

      <div v-if="totalPage > 1" class="flex items-center justify-between p-6">
        <ion-button expand="block" fill="outline" @click="prevPage"
          ><svg
            xmlns="http://www.w3.org/2000/svg"
            class="w-5 h-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            stroke-width="2"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M10 19l-7-7m0 0l7-7m-7 7h18"
            /></svg
        ></ion-button>

        <div>{{ `${currentPage}/${totalPage}` }}</div>
        <ion-button expand="block" fill="outline" @click="nextPage">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="w-5 h-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            stroke-width="2"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M14 5l7 7m0 0l-7 7m7-7H3"
            />
          </svg>
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import {
  IonPage,
  IonItem,
  IonLabel,
  IonContent,
  IonList,
  IonImg,
  IonButtons,
  IonHeader,
  IonToolbar,
  IonBackButton,
  IonTitle,
  IonButton,
  IonSearchbar,
} from '@ionic/vue'

const route = useRoute()
const router = useRouter()
const items = ref([])
const baseUrl = ref()
const totalPage = ref()
const currentPage = ref()

watch(
  () => route.params.path,
  (value) => fetchData(value)
)

function fetchData(path: string | any[]) {
  items.value = []
  if (path) {
    fetch('https://627799242f94a1d706102b3b.mockapi.io/tunnel/1')
      .then((res) => res.json())
      .then((data) => {
        baseUrl.value = data.tunnel
        fetch(`${baseUrl.value}/api/v1/files/${(path as any).join('/')}`)
          .then((res) => res.json())
          .then((data) => {
            currentPage.value = data.currentPage
            totalPage.value = data.totalPage
            items.value = data.items
          })
      })
  } else {
    fetch('https://627799242f94a1d706102b3b.mockapi.io/tunnel/1')
      .then((res) => res.json())
      .then((data) => {
        baseUrl.value = data.tunnel
        fetch(`${baseUrl.value}/api/v1/files/`)
          .then((res) => res.json())
          .then((data) => {
            currentPage.value = data.currentPage
            totalPage.value = data.totalPage
            items.value = data.items
          })
      })
  }
}

function navigate(item: string) {
  if (route.params.path) {
    router.push('/' + (route.params.path as any).join('/') + '/' + item)
  } else {
    router.push('/' + item)
  }
}

function isImage(url: string) {
  return /\.(jpg|jpeg|png|webp|avif|gif|svg)$/.test(url)
}

function getImage(item: string) {
  if (isImage(item)) {
    return `${baseUrl.value}/api/v1/files/${(route.params.path as any).join(
      '/'
    )}/${item}`
  }
}

function nextPage() {
  if (currentPage.value < totalPage.value) {
    items.value = []
    currentPage.value += 1
    fetch(
      `${baseUrl.value}/api/v1/files/${(route.params.path as any).join(
        '/'
      )}?page=${currentPage.value}`
    )
      .then((res) => res.json())
      .then((data) => {
        currentPage.value = data.currentPage
        totalPage.value = data.totalPage
        items.value = data.items
      })
  }
}

function prevPage() {
  if (currentPage.value > 1) {
    items.value = []
    currentPage.value -= 1
    fetch(
      `${baseUrl.value}/api/v1/files/${(route.params.path as any).join(
        '/'
      )}?page=${currentPage.value}`
    )
      .then((res) => res.json())
      .then((data) => {
        currentPage.value = data.currentPage
        totalPage.value = data.totalPage
        items.value = data.items
      })
  }
}

function onSearchChange(event: any) {
  if (event.target.value === '') {
    fetchData(route.params.path)
  } else {
    items.value = items.value.filter(
      (item) =>
        (item as string)
          .toLowerCase()
          .indexOf(event.target.value.toLowerCase()) !== -1
    )
  }
}

fetchData(route.params.path)
</script>

<style scoped>
ion-item {
  --padding-start: 0;
  --inner-padding-end: 0;
}
ion-img::part(image) {
  @apply rounded-sm w-full h-auto;
}
</style>
