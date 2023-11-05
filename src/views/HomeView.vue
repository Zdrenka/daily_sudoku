<template>
  <div class="home">
    <Sudoku msg="" />
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import Sudoku from "@/components/Sudoku.vue"; // @ is an alias to /src

@Options({
  components: {
    Sudoku,
  },
})
export default class HomeView extends Vue {}
</script>
