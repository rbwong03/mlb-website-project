<template>
  <div class="team-card">
    <div class="team-header">
      <h2 class="team-name">{{ teamName }}</h2>
    </div>
    <div class="team-body">
      <div class="team-statistics">
        <p>Record: {{ teamRecord }}</p>
        <p>Streak: {{ teamStreak }}</p>
        <p>Rank: {{ teamRank }}</p>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";

export default defineComponent({
  name: "TeamCard",
  props: {
    teamName: {
      type: String as PropType<string>,
      required: true,
      default: null,
    },
    teamRecord: {
      type: String as PropType<string>,
      required: true,
      default: null,
    },
    teamStreak: {
      type: Number as PropType<number>,
      required: true,
      default: 0,
    },
    teamRank: {
      type: Number as PropType<number>,
      required: true,
      default: 0,
    },
  },
});
</script>

<style scoped>
.team-card {
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px;
  background-color: white;
  margin: 10px;
  width: 250px;
  font-size: 1.2em;
}

.team-header {
  background-color: #081d4d;
  color: white;
  padding: 10px;
  text-align: center;
}

.team-name {
  margin: 0;
  font-size: 1.4em;
}

.team-body {
  padding: 10px;
}

.team-record,
.team-statistics p {
  margin: 5px 0;
  font-size: 1em;
  color: #333;
  font-weight: bold;
}
</style>
