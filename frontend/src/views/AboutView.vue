<template>
  <div class="about">
    <!-- <ProjetForm /> -->
    <ParticipationProjet/>

  </div>
</template>

<script setup>
import ProjetForm from "@/components/ProjetForm.vue";
import ParticipationProjet from "@/components/ParticipationProjet.vue";
</script>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
