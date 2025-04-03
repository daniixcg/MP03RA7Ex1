  <template>
  <div>
  <section v-if="currentScreen === 'setup'" class="setup-container">
      <h2 class="setup-title">Configuració dels Jugadors</h2>
      <p class="setup-instruccions">
          Introdueix els noms dels jugadors per començar el joc.
      </p>
  
      <div class="toggle-container">
          <label for="two-players-toggle">Dos Jugadors:</label>
          <label class="switch">
          <input type="checkbox" v-model="isTwoPlayers" />
          <span class="slider round"></span>
          </label>
      </div>
  
      <div class="player-input-group">
          <label for="player1-name" class="player-label">Nom del Jugador 1:</label>
          <input type="text" v-model="player1Name" class="player-input" required />
      </div>
  
      <div class="player-input-group" v-if="isTwoPlayers">
          <label for="player2-name" class="player-label">Nom del Jugador 2:</label>
          <input type="text" v-model="player2Name" class="player-input" required />
      </div>
  
      <button @click="startGame" class="setup-button">Següent</button>
  </section>
        <!-- Secció de selecció de l'equip -->
  <section v-if="currentScreen === 'teamSelection'" id="team-selection-section">
      <h2>Selecciona el teu Equip</h2>
      <h2>{{ currentPlayerSelectionMessage }}</h2>        
      <h2 id="credits-display">
          Crèdits restants: <span id="credits-value">{{ creditsDisplay }}</span>
      </h2>
      <div id="team-section">
          <h2 id="current-player-selection">{{ currentPlayerSelectionDisplay }}</h2>
          <div id="selected-team-grid" class="grid-container" ref="teamContainer">
            <pokemon-card
            v-for="(poke, index) in currentPlayerTeam"
            :key="index"
            :pokemon="poke"
            :is-selected="isPokemonInTeam(poke.name)"
            @toggle-selection="handleToggleSelection"
            />
          </div>
      </div>

      <button id="next-player-button" @click="handleNextPlayer">
          {{buttonLabel}}
      </button>
      <!-- Opcions d'ordenació -->
      <div id="sort-options-section">
          <h2>Opcions d'Ordenació</h2>
          <form id="sort-options-form">
              <fieldset>
                  <legend>Ordena per:</legend>
                  <label>
                  <input type="radio" name="sort-criteria" value="name" v-model="sortCriteria" />
                  Nom
                  </label>
                  <label>
                  <input type="radio" name="sort-criteria" value="points" v-model="sortCriteria" />
                  Punts
                  </label>
                  <label>
                  <input type="radio" name="sort-criteria" value="type" v-model="sortCriteria" />
                  Tipus
                  </label>
              </fieldset>
              <fieldset>
                  <legend>Mètode d'ordenació:</legend>
                  <label>
                  <input type="radio" name="sort-method" value="bubble" v-model="sortMethod" />
                  Bombolla
                  </label>
                  <label>
                  <input type="radio" name="sort-method" value="insertion" v-model="sortMethod" />
                  Inserció
                  </label>
                  <label>
                  <input type="radio" name="sort-method" value="selection" v-model="sortMethod" />
                  Selecció
                  </label>
              </fieldset>
              <button type="button" id="sort-team" @click="handleSortOptions">
              Ordenar
              </button>
          </form>
      </div>
      <div id="pokemon-grid" class="grid-container" ref="gridContainer">
          <pokemon-card
          v-for="(poke, index) in globalPokemonList"
          :key="index"
          :pokemon="poke"
          :is-selected="isPokemonInTeam(poke.name)"
          @toggle-selection="handleToggleSelection"
          />
      </div>
  </section>
  <section v-if="currentScreen === 'battleSection'" id="teams-overview-section">
    <h2>Vista General dels Equips</h2>
    <h3 id="player1-team-name">Equip del Jugador 1</h3>
    <div ref="player1List" id="player1-team-display" class="player1-selected-team-grid">
    <pokemon-card
    v-for="(poke, index) in player1Team"
    :key="index"
    :pokemon="poke"
    :is-selected="isPokemonInTeam(poke.name)"
    />
    </div>
    <h3 id="player2-team-name">Equip del Jugador 2</h3>
    <div ref="player2List" id="player2-team-display" class="player2-selected-team-grid">
    <pokemon-card
          v-for="(poke, index) in player2Team"
          :key="index"
          :pokemon="poke"
          :is-selected="isPokemonInTeam(poke.name)"
          />
    </div>
    <button @click="startBattle" id="start-battle-button">Iniciar la Batalla</button>

    <div class="battle-container">
      <!-- Pokémon 1 -->
      <div class="cajaGlobal">
        <div id="pokemon1-display" class="pokemon-fighter">
          <pokemon-card
            v-if = "selectedPokemon1"
            :pokemon = "selectedPokemon1"
          />
        </div>
      </div>

      <!-- VS Text -->
      <p class="vs-text">VS</p>

      <!-- Pokémon 2 -->
      <div id="pokemon2-display" class="pokemon-fighter">
        <pokemon-card
          v-if = "selectedPokemon2"
          :pokemon = "selectedPokemon2"
        />
      </div>
      <!-- Battle Log -->
      <div class="battle-log-container">
        <h2>Registre de la Batalla</h2>
        <div id="battle-log">
          <div v-for="(entry,index) in battleLogElement" key="index">
            <h2 v-if="entry.type == 'h2'">{{entry.message}}</h2>
            <p v-else-if="bold == false">{{entry.message}}</p>
            <p v-else><b>{{entry.message}}</b></p>
          </div>
        </div>
      </div>
    </div>
  </section>
</div>
</template>

<script>
import { PokemonTeamViewModel } from "./components/viewModel.js";
import PokemonCard from "./components/PokemonCard.vue";

export default {
  components: {
    "pokemon-card": PokemonCard,
  },
  data() {
    return {
      currentScreen: "setup",
      isTwoPlayers: true,
      player1Name: "",
      player2Name: "",
      currentPlayerSelectionMessage: "",
      currentPlayerSelectionDisplay: "",
      player1List: "",
      player2List: "",
      // creditsDisplay: "",
      sortCriteria: "",
      sortMethod: "",
      globalPokemonList: [],
      //      currentPlayerTeam: [],
      showTeamList: [],
      buttonLabel: "Següent Jugador",
      viewModel: new PokemonTeamViewModel(),
    };
  },
  methods: {
    startGame() {
      //      this.bindEvents();
      if (!this.player1Name || (this.isTwoPlayers && !this.player2Name)) {
        alert("Si us plau, introdueix els noms de tots els jugadors.");
        return;
      }
      if (!this.isTwoPlayers) {
        this.player2Name = "CPU";
      }
      console.log(
        `Jugador 1: ${this.player1Name}, Jugador 2: ${this.player2Name}`
      );
      this.currentScreen = "teamSelection";
      this.creditsDisplay = this.viewModel.currentPlayer.team.credits;
      this.startTeamSelection();
    },
    startTeamSelection() {
      // Call initializeMatch() on the ViewModel to set up players
      this.viewModel.initializeMatch(this.player1Name, this.player2Name);
      // Set up for Player 1's team selection
      this.viewModel.currentPlayer = this.viewModel.player1;
      this.currentPlayerSelectionMessage = `${this.player1Name}, selecciona el teu equip Pokémon`;
      this.renderGlobalList();
      //      this.updateCreditsDisplay();
    },
    // Exemple del mètode adaptat
    renderGlobalList() {
      // En lloc de manipular el DOM, actualitzem la propietat reactiva:
      this.globalPokemonList = this.viewModel.getGlobalList();
      // Això farà que Vue re-renderitzi la graella amb la nova llista.
    },
    renderSelectionTeam() {
      this.currentPlayerTeam = this.viewModel.currentPlayer.getTeam();
    },
    handleNextPlayer() {
      if (this.buttonLabel === "Fi de la selecció d'equips"){
        this.currentScreen = "battleSection";
      }
      // Check the current player and handle transitions.
      if (this.viewModel.currentPlayer === this.viewModel.player1) {
        // Player 1 finished selecting their team.
        this.viewModel.switchPlayer();
        if (this.isTwoPlayers) {
          //
          // Two-player mode: move to Player 2.
          //this.viewModel.currentPlayer = this.player2Name;
          //          this.viewModel.switchPlayer();
          //const player2Name = this.viewModel.currentPlayer.getName(); // Fetch Player 2 name
          this.currentPlayerSelectionMessage = `${this.player2Name}, selecciona el teu Pokémon`;
          //          this.updateCurrentPlayerTeam();
          // Re-render for Player 2's team selection.
          //          this.renderGlobalList();
          //          this.updateCreditsDisplay();
          this.buttonLabel = "Fi de la selecció d'equips";
        } else {
          // One player vs. CPU: automatically select a team for the CPU.
          this.currentPlayerSelectionMessage = `${this.player2Name} ha seleccionat el seu equip.`;
          this.viewModel.autoSelectCpuTeam();
          //          this.updateCurrentPlayerTeam();
          this.buttonLabel = "Fi de la selecció d'equips";
          //          this.transitionToBattle();
        }
      } else if (this.viewModel.currentPlayer === this.viewModel.player2) {
        // Both players have selected their teams.
      }
    },
    updateCurrentPlayerTeam() {
      // En lloc de manipular el DOM, actualitzem la propietat reactiva:
      this.currentPlayerTeam = this.viewModel.getCurrentTeam();
      // Això farà que Vue re-renderitzi la graella amb la nova llista.
    },
    handleSortOptions() {
      this.viewModel.sortGlobalList(this.sortCriteria, this.sortMethod);
      this.renderGlobalList();
    },
    updateCreditsDisplay() {
      this.creditsDisplay = `${this.viewModel
        .getCurrentPlayer()
        .getCredits()}`;
    },
    isPokemonInTeam(name) {
      const playerTeam =
        this.viewModel.currentPlayer === this.viewModel.player1
          ? this.viewModel.player1.team
          : this.viewModel.player2.team;
      return playerTeam.selectedTeam.some((p) => p.name === name);
    },
    // Called from inside the parent when child emits `toggle-selection`
    handleToggleSelection(pokemon) {
      // If it's already in the team, remove it. Otherwise, try to add it.
      const isInTeam = this.isPokemonInTeam(pokemon.name);
      if (isInTeam) {
        this.viewModel.removePokemonFromTeam(pokemon.name);
      } else {
        const addResult =
          this.viewModel.addPokemonToCurrentPlayer(pokemon);
        if (!addResult) {
          alert("No es pot afegir el Pokémon.");
        }
      }

      // After adding/removing, refresh the player's team
      //      this.updateCurrentPlayerTeam();
      // Possibly update credits or other UI
      //      this.updateCreditsDisplay();
    },
    startBattle() {
      console.log("🔥 Battle started!");
      
      // Show the teams display and battle arena
      document.getElementById("player1-team-name").textContent = this.viewModel.player1.getName();
      document.getElementById("player2-team-name").textContent = this.viewModel.player2.getName();
  
      // Call ViewModel to start battle
      this.viewModel.startBattle();
    },
  },
  // Altres mètodes...
  mounted() {
    this.viewModel.fetchAndLoadPokemons();
    // Aquí pots comprovar la referència del grid si cal
    console.log("Grid container:", this.$refs.gridContainer);
  },
  // ...
  computed: {
    creditsDisplay() {
      // Return the current player’s credits directly from the viewModel
      return this.viewModel.currentPlayer.team.credits;
    },
    currentPlayerTeam() {
      return this.viewModel.getCurrentTeam();
    },
    player1Team(){
      return this.viewModel.player1.getTeam();
    },
    player2Team(){
      return this.viewModel.player2.getTeam();
    },
    selectedPokemon1(){
      return this.viewModel.getWhoIsFighting("Player1");
    },
    selectedPokemon2(){
      return this.viewModel.getWhoIsFighting("Player2");
    },
    battleLogElement(){
        return this.viewModel.callBattleLog();
    }
  },
};
</script>
  