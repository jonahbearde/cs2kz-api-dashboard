<template>
  <div class="font-poppings text-slate-200">
    <n-config-provider :theme="darkTheme" :locale="enUS">
      <n-message-provider>
        <Header :loading="loading" />
        <div class="flex">
          <NavBar />
          <div class="flex-1 bg-gray-900 p-4">
            <div class="flex mb-4">
              <div
                class="flex items-center mr-1"
                v-for="(part, index) in pathArray"
                :key="index"
              >
                <RouterLink
                  class="mr-1"
                  :class="
                    index !== pathArray.length - 1
                      ? 'bg-gray-800 hover:bg-gray-700 text-blue-600 py-1 px-[10px] rounded-md'
                      : 'cursor-default'
                  "
                  :to="getLink(index)"
                  >{{ part }}
                </RouterLink>
                <ion-icon
                  v-if="index !== pathArray.length - 1"
                  size=""
                  name="chevron-forward-sharp"
                ></ion-icon>
              </div>
            </div>
            <RouterView />
          </div>
        </div>
      </n-message-provider>
    </n-config-provider>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue"
import Header from "./components/Header.vue"
import NavBar from "./components/NavBar.vue"
import { RouterView, useRoute } from "vue-router"
import { onBeforeMount, computed } from "vue"
import { darkTheme, NConfigProvider, NMessageProvider, enUS } from "naive-ui"
import { useAdminStore } from "./store/admin"
import axiosClient from "./axios"

const adminStore = useAdminStore()
const route = useRoute()
const loading = ref(false)

onBeforeMount(async () => {
  loading.value = true
  const steamId = getCookie("steam_id")
  // console.log('steamId', steamId);

  if (steamId) {
    try {
      const { data } = await axiosClient.get(
        `/players/${encodeURIComponent(steamId)}/steam`
      )
      // console.log(data)

      if (data) {
        adminStore.steamId = data.steam_id
        adminStore.name = data.username
        adminStore.avatar_url = data.avatar_url
      }
    } catch (error) {
      console.log(error)
    }
  }

  loading.value = false
})

const pathArray = computed(() =>
  route.path
    .split("/")
    .filter((item) => item !== "")
    .map((item) => item.charAt(0).toUpperCase() + item.slice(1))
)

function getLink(index: number) {
  const path = pathArray.value
    .slice(0, index + 1)
    .join("/")
    .toLowerCase()
  return "/" + path
}

function getCookie(name: string) {
  let matches = document.cookie.match(
    new RegExp(
      "(?:^|; )" +
        name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, "\\$1") +
        "=([^;]*)"
    )
  )
  return matches ? decodeURIComponent(matches[1]) : undefined
}
</script>

<style scoped>
ion-icon {
  font-size: 32;
}
</style>
