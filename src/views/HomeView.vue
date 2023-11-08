<template>
  <div class="division-rankings">
    <h1>Division Rankings</h1>
    <div class="filters">
      <div class="date-filter">
        <label for="dateFilter">Games on date:</label>
        <input type="date" v-model="selectedDate" />
      </div>
      <div class="button-group">
        <button @click="sortTeamsByRank">
          {{
            sortedByRank ? "Rank Lowest To Highest" : "Rank Highest to Lowest"
          }}
        </button>
        <button @click="sortTeamsByStreak">
          {{
            sortedByStreak
              ? "Streak Lowest to Highest"
              : "Streak Highest to Lowest"
          }}
        </button>
      </div>
    </div>
    <div class="divisions-container">
      <div
        v-for="(division, divisionName) in divisionsData"
        :key="divisionName"
        class="division-column"
      >
        <h2>{{ divisionName.replace(/_/g, " ") }}</h2>
        <transition-group name="team" tag="div" class="team-cards-container">
          <div
            v-for="(teamData, teamName) in division.teams"
            :key="teamData.sortKey"
          >
            <team-card
              v-if="teamData.gameToShow"
              :team-name="teamName"
              :team-record="teamData.gameToShow.Record"
              :team-streak="
                teamData.gameToShow.Streak !== null
                  ? teamData.gameToShow.Streak
                  : 0
              "
              :team-rank="teamData.gameToShow.Rank"
            />
            <div v-else>No recent games available for {{ teamName }}.</div>
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, watch, computed } from "vue";
import TeamCard from "@/components/UserCard.vue";
import TeamGame from "@/types/TeamGame";

export default defineComponent({
  name: "DivisionList",
  components: {
    TeamCard,
  },
  setup() {
    const selectedDate = ref<string>("2023-10-01");
    const year = computed(() => selectedDate.value.split("-")[0]);
    const sortedByRank = ref(false);
    const sortedByStreak = ref(false);

    const updateSortKeys = (teamsArray: any[], sortBy: any[]) => {
      teamsArray.forEach((team, index) => {
        team[1].sortKey =
          sortBy === team.Rank ? index : teamsArray.length - index - 1;
      });
    };

    const sortTeamsByRank = () => {
      sortedByRank.value = !sortedByRank.value;
      // Ensure that sorting by streak is disabled when sorting by rank
      sortedByStreak.value = false;

      Object.values(divisionsData).forEach((division) => {
        const teamsArray = Object.entries(division.teams);
        teamsArray.sort((a, b) => {
          const rankA = a[1].gameToShow
            ? a[1].gameToShow.Rank
            : Number.MAX_VALUE;
          const rankB = b[1].gameToShow
            ? b[1].gameToShow.Rank
            : Number.MAX_VALUE;
          return sortedByRank.value ? rankA - rankB : rankB - rankA;
        });

        updateSortKeys(teamsArray, "rank");
        division.teams = Object.fromEntries(teamsArray);
      });
    };

    const sortTeamsByStreak = () => {
      sortedByStreak.value = !sortedByStreak.value;
      sortedByRank.value = false;

      Object.values(divisionsData).forEach((division) => {
        const teamsArray = Object.entries(division.teams);
        teamsArray.sort((a, b) => {
          const streakA = a[1].gameToShow ? a[1].gameToShow.Streak || 0 : 0;
          const streakB = b[1].gameToShow ? b[1].gameToShow.Streak || 0 : 0;
          return sortedByStreak.value ? streakB - streakA : streakA - streakB;
        });

        updateSortKeys(teamsArray, "streak");
        division.teams = Object.fromEntries(teamsArray);
      });
    };

    const allTeamsData = reactive<{
      [teamName: string]: { games: TeamGame[]; gameToShow: TeamGame | null };
    }>({});
    const divisionsData = reactive<{
      [divisionName: string]: {
        division: string;
        teams: {
          [teamName: string]: {
            sortKey: number;
            games: TeamGame[];
            gameToShow: TeamGame | null;
          };
        };
      };
    }>({});

    const findMostRecentGame = (
      teamGames: TeamGame[],
      date: string
    ): TeamGame | null => {
      const gamesBeforeDate = teamGames.filter((game) => game.Date < date);
      if (gamesBeforeDate.length === 0) return null;
      gamesBeforeDate.sort((a, b) => (b.Date < a.Date ? -1 : 1));
      return gamesBeforeDate[0];
    };

    const findGameToShow = (
      teamGames: TeamGame[],
      date: string
    ): TeamGame | null => {
      const gameOnDate = teamGames.find((game) => game.Date === date);
      return gameOnDate || findMostRecentGame(teamGames, date);
    };

    const loadGameData = async (year: string) => {
      const divisions = [
        "NL_CENTRAL",
        "NL_EAST",
        "NL_WEST",
        "AL_CENTRAL",
        "AL_EAST",
        "AL_WEST",
      ];

      for (const division of divisions) {
        const teamNames = getTeamNamesByDivision(division);
        if (!divisionsData[division]) {
          divisionsData[division] = { division: division, teams: {} };
        }

        for (const teamName of teamNames) {
          try {
            const module = await import(
              `@/MLB_ALL_DATA_JSON/year${year}/${division}/${teamName}_${year}.json`
            );

            const games = module.default.team_games as TeamGame[];
            allTeamsData[teamName] = { games, gameToShow: null };
            divisionsData[division].teams[teamName] = {
              games,
              gameToShow: null,
            };

            allTeamsData[teamName].gameToShow = findGameToShow(
              games,
              selectedDate.value
            );
          } catch (error) {
            console.error(
              `Error loading game data for team ${teamName}:`,
              error
            );
          }
        }
      }
    };

    function getTeamNamesByDivision(division: string): string[] {
      const divisionTeamsMap: { [key: string]: string[] } = {
        NL_CENTRAL: ["CHC", "STL", "MIL", "PIT", "CIN"],
        NL_EAST: ["ATL", "NYM", "PHI", "MIA", "WSN"],
        NL_WEST: ["LAD", "SDP", "SFG", "COL", "ARI"],
        AL_CENTRAL: ["CLE", "KCR", "DET", "MIN", "CHW"],
        AL_EAST: ["NYY", "TBR", "BOS", "TOR", "BAL"],
        AL_WEST: ["HOU", "OAK", "SEA", "TEX", "LAA"],
      };
      return divisionTeamsMap[division] || [];
    }

    watch(
      selectedDate,
      async (newDate) => {
        const newYear = newDate.split("-")[0];

        Object.keys(allTeamsData).forEach((key) => {
          delete allTeamsData[key];
        });

        await loadGameData(newYear);

        Object.entries(allTeamsData).forEach(([teamName, teamData]) => {
          const gameToShow = findGameToShow(teamData.games, newDate);
          allTeamsData[teamName].gameToShow = gameToShow;
        });
        Object.values(divisionsData).forEach((division) => {
          Object.entries(division.teams).forEach(([teamName, teamData]) => {
            const gameToShow = findGameToShow(teamData.games, newDate);
            teamData.gameToShow = gameToShow;
          });
        });
      },
      { immediate: true }
    );
    return {
      selectedDate,
      allTeamsData,
      year,
      divisionsData,
      sortedByRank,
      sortedByStreak,
      sortTeamsByRank,
      sortTeamsByStreak,
    };
  },
});
</script>
<style scoped>
.divisions-container {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  padding: 20px;
}

.division-column {
  flex: 1;
  min-width: 250px;
  max-width: 350px;
  border: 1px solid #081d4d;
  border-radius: 8px;
  padding: 10px;
  background-color: #f9f9f9;
  margin-bottom: 20px;
}

.team-cards-container {
  display: flex;
  flex-direction: column;
}

button {
  padding: 10px 15px;
  border: none;
  border-radius: 10px;
  background-color: #081d4d;
  cursor: pointer;
  font-weight: bold;
  color: white;
}

button:hover {
  background-color: #d50032;
}

.filters {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.date-filter {
  text-align: center;
  margin-bottom: 15px;
}

.button-group {
  display: flex;
  justify-content: center;
  gap: 10px;
}
.division-column h2 {
  background-color: #d50032;
  color: white;
  padding: 10px;
  border-radius: 8px;
  text-align: center;
}
.team-move {
  transition: all 1s;
}
</style>
