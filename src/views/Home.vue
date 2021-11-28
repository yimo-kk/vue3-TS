<template>
  <div class="home">
    <cProgress msg="Welcome to Your Vue.js + TypeScript App" />
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import cProgress from "@/components/c-progress.vue"; // @ is an alias to /src

@Options({
  components: {
    cProgress,
  },
})
export default class Home extends Vue {}
</script>
