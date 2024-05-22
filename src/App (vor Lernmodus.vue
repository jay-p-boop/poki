<template>
  <div id="app">
        <button class="floating-mic" @click="listenToPokemonName">🎤</button>

    <div class="pokemon-row" v-for="evolutionLine in evolutionLines" :key="evolutionLine[0].name">
      <div class="pokemon-card" v-for="pokemon in evolutionLine" :key="pokemon.name">
        <img :src="pokemon.image" @click="speakPokemonName(pokemon.name)" alt="Pokemon Image" />
        <p>{{ pokemon.name }}</p>
      </div>
    </div>
  </div>
</template>

<style>
#app {
  text-align: center;
  background-color: #f6f6f6;  
  padding-top: 20px;
}

.pokemon-row {
  display: flex;
  flex-direction: column;  /* Ändern Sie die Anordnung zu einer Spalte */
  align-items: center;  /* Zentriert die Karten in der Mitte */
  margin-bottom: 30px;  
}

.pokemon-card {
  border: 2px solid #ddd;  
  margin-bottom: 20px;  
  padding: 30px;  
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;  
  color: black;
  margin-top: 10px;
}

.pokemon-card img {
  width: 200px;  
  height: 200px;  
  object-fit: cover;
  margin-bottom: 10px;  
  
}
 

.pokemon-row{
  background-color: #e9e9e9; /* Ein etwas dunklerer Grauton für gerade Reihen */
      margin-top: 15px;/* Ein heller Grauton für ungerade Reihen */

}
  .floating-mic {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000; /* Stellen Sie sicher, dass der Button immer im Vordergrund ist */
  background-color: #F44336; /* Roter Hintergrund */
  color: white; /* Weißer Text */
  border: none;
  border-radius: 50%; /* Kreisförmiger Button */
  width: 60px;
  height: 60px;
  font-size: 24px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.3); /* Ein bisschen Schatten für den 3D-Effekt */
}
</style>



<script>
  import levenshtein from 'fast-levenshtein';

import pokemonsData from './pokemons.json'; // Pfad zur pokemons.json Datei

export default {
  data() {
    return {
      pokemons: pokemonsData,
      evolutionLines: []
    };
  },
  created() {
    this.createEvolutionLines();
  },
  methods: {
    createEvolutionLines() {
      let lines = {};
      
      this.pokemons.forEach(pokemon => {
        const lineName = pokemon.evolutionLine.join('-');
        if (!lines[lineName]) {
          lines[lineName] = [];
        }
        lines[lineName].push(pokemon);
      });

      this.evolutionLines = Object.values(lines);
    },
    speakPokemonName(name) {
      const speech = new SpeechSynthesisUtterance(name);
      window.speechSynthesis.speak(speech);
    },
    listenToPokemonName() {
  const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition || window.mozSpeechRecognition || window.msSpeechRecognition)();
  recognition.lang = 'de-DE'; // Set to German
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;
  recognition.start();

recognition.onresult = (event) => {
    const spokenName = event.results[0][0].transcript.toLowerCase().trim();
    let found = false;
    let bestMatchIndex = null;
    let minDistance = Infinity;

    for (let i = 0; i < this.evolutionLines.length; i++) {
      for (let j = 0; j < this.evolutionLines[i].length; j++) {
        const currentPokemonName = this.evolutionLines[i][j].name.toLowerCase();
        const distance = levenshtein.get(spokenName, currentPokemonName);
        
        if (distance < minDistance) {
          minDistance = distance;
          bestMatchIndex = i;
        }

        if (currentPokemonName === spokenName) {
          found = true;
          break;
        }
      }
      if (found) break;
    }

    if (found) {
      const el = document.querySelectorAll('.pokemon-row')[bestMatchIndex];
      el.scrollIntoView({ behavior: 'smooth', block: 'start' });
    } else if (minDistance <= 2) { // Wenn eine ähnliche Übereinstimmung gefunden wurde
      const el = document.querySelectorAll('.pokemon-row')[bestMatchIndex];
      el.scrollIntoView({ behavior: 'smooth', block: 'start' });
      alert(`Meinten Sie "${this.evolutionLines[bestMatchIndex][0].name}"?`);
    } else {
      alert(`Kein Pokémon namens "${spokenName}" gefunden.`);
    }
};
;

  recognition.onerror = (event) => {
    alert('Ein Fehler ist aufgetreten. Bitte versuchen Sie es erneut.');
  };
}
,
  },
};
</script>