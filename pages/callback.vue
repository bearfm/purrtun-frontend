<template>
  <h1 class="text-black">{{ response }}</h1>
</template>

<script setup lang="ts">
import { useCookie, useRuntimeConfig, navigateTo, useRoute } from '#imports';

const accessTokenCookie = useCookie("token", { sameSite: true, maxAge: 60 * 60 * 24 });
let response = ref(null);

const conf = useRuntimeConfig();
const backend = conf.public.backend;

fetch(backend + "/auth", {
  method: "POST",
  body: JSON.stringify({
    "accessToken": `${useRoute().query.code}`,
    "self": `${String(useRoute().name)}`
  }),
  headers: {
    'Content-Type': 'application/json',
  },
  credentials: "include"
}).then((res) => res.json())
    .then((data) => {
      console.log(data);
      if (!data.encryptedToken) {
        throw new Error("No accessToken returned from the API: " + JSON.stringify(data));
      }
      accessTokenCookie.value = data.encryptedToken;

      if (!(accessTokenCookie.value === data.encryptedToken)) {
        throw new Error("For some reason, your browser failed to properly set the cookie.");
      }

      response.value = data;
      navigateTo("/");
    })
    .catch((error) => {
      console.error('Error:', error);
      response.value = { error: error.message };
    });
</script>
