<template>
  <div class="game-schedule">
    <h1>Game Schedule</h1>
    <div class="filters">
      <div class="date-filter">
        <label for="dateFilter">Games on date:</label>
        <input type="date" id="dateFilter" v-model="dateFilter" />
      </div>
      <div class="additional-filters">
        <div class="attendance-filter">
          <label for="attendanceThreshold"
            >Games with Attendance less than :</label
          >
          <input
            type="number"
            id="attendanceThreshold"
            v-model.number="attendanceThreshold"
          />
        </div>
        <button @click="toggleScoreSorting" class="score-sorting-button">
          {{ sortScores ? "Sort by Lowest Score" : "Sort by Highest Score" }}
        </button>
      </div>
    </div>
    <div class="game-header">
      <span class="game-date-header">Date</span>
      <span class="game-teams-header">Teams</span>
      <span class="game-score-header">Score</span>
      <span class="game-attendance-header">Attendance</span>
    </div>

    <transition-group name="list" tag="ul">
      <li
        v-for="game in filteredGames"
        :key="game.Unnamed_0"
        class="game-entry"
      >
        <span class="game-date">
          {{
            game.Date
              ? new Date(addDayToDate(game.Date)).toLocaleDateString()
              : "Invalid date"
          }}
        </span>

        <span class="game-match">
          <img
            :src="getTeamLogo(game.Away_Team)"
            class="team-logo"
            alt="Away Team Logo"
          />
          {{ game.Away_Team }} @
          {{ game.Home_Team }}
          <img
            :src="getTeamLogo(game.Home_Team)"
            class="team-logo"
            alt="Home Team Logo"
          />
        </span>
        <span class="game-score"
          >{{ game.Away_Team }}: {{ game.Away_Runs }}, {{ game.Home_Team }}:
          {{ game.Home_Runs }}</span
        >
        <span class="game-attendance">{{ game.Attendance }}</span>
      </li>
      <li v-if="filteredGames.length === 0" class="no-games-message">
        No games on this date.
      </li>
    </transition-group>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch } from "vue";
import Game from "../types/Game";

export default defineComponent({
  name: "GameView",
  setup() {
    const games = ref<Game[]>([]);
    const attendanceThreshold = ref<number | null>(50000);
    const dateFilter = ref<string>("2023-10-01");
    const sortScores = ref<boolean>(false);
    const sortDirection = ref<"ascending" | "descending">("descending");

    const addDayToDate = (dateString: string | number | Date) => {
      const date = new Date(dateString);
      date.setDate(date.getDate() + 1);
      return date.toISOString().split("T")[0];
    };

    const getTeamLogo = (teamName: string): string => {
      return require(`@/assets/${teamName}.png`);
    };

    const toggleScoreSorting = () => {
      sortScores.value = !sortScores.value;
      sortDirection.value = sortScores.value ? "descending" : "ascending";
    };

    const hasGamesOnSelectedDate = computed(() => {
      return filteredGames.value.length > 0;
    });

    const filteredGames = computed(() => {
      let filtered = games.value
        .filter((game) => {
          return (
            !attendanceThreshold.value ||
            game.Attendance < attendanceThreshold.value
          );
        })
        .filter((game) => {
          return !dateFilter.value || game.Date === dateFilter.value;
        });

      if (sortDirection.value === "descending") {
        filtered.sort(
          (a, b) => b.Away_Runs + b.Home_Runs - (a.Away_Runs + a.Home_Runs)
        );
      } else {
        filtered.sort(
          (a, b) => a.Away_Runs + a.Home_Runs - (b.Away_Runs + b.Home_Runs)
        );
      }

      return filtered;
    });

    watch(
      dateFilter,
      async (newDate, oldDate) => {
        if (newDate !== oldDate) {
          try {
            const year = newDate.slice(0, 4);
            const data = await import(
              `@/MLB_ALL_DATA_JSON/unique_games_per_season/all_unique_games_in_${year}.json`
            );
            games.value = data.default.games.map((game: Game) => ({
              ...game,
              Attendance: game.Attendance ?? 0,
            }));
          } catch (error) {
            console.error("Error loading data:", error);
          }
        }
      },
      { immediate: true }
    );
    return {
      games,
      filteredGames,
      getTeamLogo,
      attendanceThreshold,
      dateFilter,
      toggleScoreSorting,
      sortScores,
      addDayToDate,
      hasGamesOnSelectedDate,
      sortDirection,
    };
  },
});
</script>

<style scoped>
.game-schedule {
  max-width: 1000px;
  margin: 10px auto;
  display: flex;
  flex-direction: column;
}

.filters {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.game-entry {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 5px;
  border-radius: 10px;
  background-color: #fffdd8;
  box-shadow: 2px 2px 4px 0;
}
.game-header {
  margin-left: 100000px;
}

.game-header,
.game-entry {
  box-sizing: border-box;
  display: flex;
  justify-content: space-between;
  padding: 10px;
  margin-bottom: 5px;
  border-radius: 10px;
}
.game-header {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  margin-bottom: 5px;
  background-color: #d50032;
  border-radius: 10px;
  font-weight: bold;
  color: white;
}
.game-header,
.game-score,
.game-match,
.game-date {
  flex-grow: 1;
  text-align: left;
  margin: 10px 4px;
  font-weight: bold;
}
.game-attendance {
  flex: 0 0 auto;
  text-align: right;
  margin-left: auto;
  font-weight: bold;
}

.team-logo {
  width: 30px;
  height: auto;
  margin-right: 5px;
}

.list-move {
  transition: all 1s;
}
button {
  padding: 10px 15px;
  border: 1px solid #000;
  border-radius: 10px;
  background-color: #081d4d;
  cursor: pointer;
  font-weight: bold;
  color: white;
}

button:hover {
  background-color: #d50032;
}
.no-games-message {
  list-style-type: none;
}
.filters {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.date-filter {
  margin-bottom: 15px;
}

.additional-filters {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.attendance-filter,
.score-sorting-button {
  width: 100%;
  text-align: center;
}
.game-date-header {
  margin-left: 10px;
}

.game-teams-header {
  padding-left: 70px;
}

.game-date-header {
  padding-left: 50px;
}
.game-score-header {
  padding-left: 100px;
}
</style>
