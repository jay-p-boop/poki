<template>
  <div id="app">
    <button 
      class="floating-mic" 
      :class="{ 'listening': isListening }" 
      :disabled="micButtonDisabled"
      @click="listenToPokemonName"
    >
      <div v-if="isListening" class="pulse"></div>
      🎤
    </button>
    
    
    <!-- App Bar am unteren Rand des Bildschirms -->
    <div class="app-bar">
      <button 
        class="tab-button" 
        :class="{ 'active-tab': !learningMode }" 
        @click="stopLearningMode"
      >
        🏠 Start
      </button>
      <button 
        class="tab-button" 
        :class="{ 'active-tab': learningMode }" 
        @click="startLearningMode"
      >
        💡 Lernen
      </button>
    </div>

    


    <!-- Lernmodus -->
    <div class="pokemon-card" v-if="learningMode">
      <img 
        :src="currentLearningPokemon.image" 
        :class="{ 'correct-answer': correctPokemonIndex === currentLearningIndex }" 
        @click="speakPokemonName(currentLearningPokemon.name)" 
        alt="Learning Pokemon Image" 
      />
      <p>Was ist der Name dieses Pokémon?</p>
      
      <button class="button-80" @click="showNextPokemon">➡️</button>
      <p>Erkannter Text: {{ recognizedText }}</p>
    </div>

    <!-- Standardansicht -->
    <div v-else>
      
      <div class="pokemon-row" v-for="evolutionLine in evolutionLines" :key="evolutionLine[0].name">
        <div class="pokemon-card" v-for="(pokemon, index) in evolutionLine" :key="pokemon.name" :ref="`pokemon-${pokemon.name}`">
          <img :src="pokemon.image" @click="speakPokemonName(pokemon.name)" alt="Pokemon Image" />
          <p>{{ pokemon.name }}</p>
        </div>
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
  .app-bar {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 50px;
    background-color: grey;
    display: flex;
    justify-content: space-around;
    align-items: center;
    box-shadow: 0 -2px 5px rgba(0,0,0,0.1);
    z-index: 1000;
  }

  .tab-button {
    background-color: transparent;
    color: #fff;
    border: none;
    font-size: 16px;
    font-weight: bold;
    padding: 10px;
    cursor: pointer;
    outline: none;
  }

  .active-tab {
    border-bottom: 2px solid #1899D6;
  }
  .button-19 {
    appearance: button;
    background-color: #1899D6;
    border: solid transparent;
    border-radius: 16px;
    border-width: 0 0 4px;
    box-sizing: border-box;
    color: #FFFFFF;
    cursor: pointer;
    display: inline-block;
    font-family: din-round,sans-serif;
    font-size: 15px;
    font-weight: 700;
    letter-spacing: .8px;
    line-height: 20px;
    margin: 0;
    outline: none;
    overflow: visible;
    padding: 13px 16px;
    text-align: center;
    text-transform: uppercase;
    touch-action: manipulation;
    transform: translateZ(0);
    transition: filter .2s;
    user-select: none;
    -webkit-user-select: none;
    vertical-align: middle;
    white-space: nowrap;
    width: 100%;
  }

  .button-19:after {
    background-clip: padding-box;
    background-color: #1CB0F6;
    border: solid transparent;
    border-radius: 16px;
    border-width: 0 0 4px;
    bottom: -4px;
    content: "";
    left: 0;
    position: absolute;
    right: 0;
    top: 0;
    z-index: -1;
  }

  .button-19:main,
  .button-19:focus {
    user-select: auto;
  }

  .button-19:hover:not(:disabled) {
    filter: brightness(1.1);
    -webkit-filter: brightness(1.1);
  }

  .button-19:disabled {
    cursor: auto;
  }
  
  .button-80 {
    background: #fff;
    backface-visibility: hidden;
    border-radius: .375rem;
    border-style: solid;
    border-width: .125rem;
    box-sizing: border-box;
    color: #212121;
    cursor: pointer;
    display: inline-block;
    font-family: Circular,Helvetica,sans-serif;
    font-size: 1.125rem;
    font-weight: 700;
    letter-spacing: -.01em;
    line-height: 1.3;
    padding: .875rem 1.125rem;
    position: relative;
    text-align: left;
    text-decoration: none;
    transform: translateZ(0) scale(1);
    transition: transform .2s;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
  }

  .button-80:not(:disabled):hover {
    transform: scale(1.05);
  }

  .button-80:not(:disabled):hover:active {
    transform: scale(1.05) translateY(.125rem);
  }

  .button-80:focus {
    outline: 0 solid transparent;
  }

  .button-80:focus:before {
    content: "";
    left: calc(-1*.375rem);
    pointer-events: none;
    position: absolute;
    top: calc(-1*.375rem);
    transition: border-radius;
    user-select: none;
  }

  .button-80:focus:not(:focus-visible) {
    outline: 0 solid transparent;
  }

  .button-80:focus:not(:focus-visible):before {
    border-width: 0;
  }

  .button-80:not(:disabled):active {
    transform: translateY(.125rem);
  }
  
.pokemon-row {
  display: flex;
  flex-direction: column;  /* Ändern Sie die Anordnung zu einer Spalte */
  align-items: center;  /* Zentriert die Karten in der Mitte */
  margin-bottom: 30px;  
   border-radius: 31%;
}

.pokemon-card {
  border: 2px solid grey;  
  margin-bottom: 20px;  
  padding: 30px;  
  box-shadow: 0 4px 9px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;  
  color: black;
  margin-top: 10px;
  border-radius: 5%;
}

  .pokemon-card p {
    font-size: 18px;
    font-weight: bold;
  }


.pokemon-card img {
  width: 200px;  
  height: 200px;  
  object-fit: cover;
  margin-bottom: 10px;  
  
}
  .correct-answer {
    border: 3px solid #00FF00; /* Grün */
  }

   .button-24 {
     background: #FF4742;
     border: 1px solid #FF4742;
     border-radius: 6px;
     box-shadow: rgba(0, 0, 0, 0.1) 1px 2px 4px;
     box-sizing: border-box;
     color: #FFFFFF;
     cursor: pointer;
     display: inline-block;
     font-family: nunito,roboto,proxima-nova,"proxima nova",sans-serif;
     font-size: 16px;
     font-weight: 800;
     line-height: 16px;
     min-height: 56px;
     outline: 0;
     padding: 12px 14px;
     text-align: center;
     text-rendering: geometricprecision;
     text-transform: none;
     user-select: none;
     -webkit-user-select: none;
     touch-action: manipulation;
     vertical-align: middle;
   }

   .button-24:hover,
   .button-24:active {
     background-color: initial;
     background-position: 0 0;
     color: #FF4742;
   }

   .button-24:active {
     opacity: .5;
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
  .floating-mic.listening {
  background-color: #4CAF50;  /* Grün, um zu signalisieren, dass es zuhört */
}

  .pulse {
    position: absolute;
    border: 5px solid #aaffaa; /* Leichtere grüne Farbe */
    border-radius: 50%;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    animation: pulse-animation 0.999s infinite;
  }
  body {
    font-family: 'Roboto', sans-serif;
    background-color: #f0f0f0;
    color: #333;
  }
  #app {
    background-color: #ffffff;
  }

  @keyframes pulse-animation {
    0% { transform: scale(1); opacity: 0.5; }
    100% { transform: scale(1.85); opacity: 0; }
  }

  @media (max-width: 768px) {
    .pokemon-row {
      flex-direction: column;
    }
    .pokemon-card {
      margin-bottom: 10px;
    }
  }
</style>


<script>
import levenshtein from 'fast-levenshtein';
import pokemonsData from './pokemons.json';
  import successSoundPath from '@/assets/success.mp3';
  import wrongSoundPath from '@/assets/wrong.mp3';
  import doubleMetaphonePackage from 'doublemetaphone';
  import * as SpeechSDK from 'microsoft-cognitiveservices-speech-sdk';




export default {
  data() {
    return {
      pokemons: pokemonsData,
      evolutionLines: [],
      learningMode: false,
      currentLearningPokemon: null,
      currentLearningIndex: null,
      recognizedText: "",
      successSound: new Audio(successSoundPath),
      wrongSound: new Audio(wrongSoundPath),
      correctPokemonIndex: null,
      permissionGranted: false,
      micButtonDisabled: false,
      encoder: new doubleMetaphonePackage(),
      isListening: false
    };
  },
  created() {
    this.createEvolutionLines();
    navigator.permissions.query({ name: 'microphone' }).then(permissionStatus => {
      this.permissionGranted = permissionStatus.state === 'granted';
    });
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
    speakMessage(message) {
      const speech = new SpeechSynthesisUtterance(message);
      window.speechSynthesis.speak(speech);
    },
    speakPokemonName(name) {
      this.speakMessage(name);
    },
    startLearningMode() {
      this.currentLearningIndex = Math.floor(Math.random() * this.pokemons.length);
      this.currentLearningPokemon = this.pokemons[this.currentLearningIndex];
      this.learningMode = true;
    },
    stopLearningMode() {
      this.learningMode = false;
      this.currentLearningPokemon = null;
      this.currentLearningIndex = null;
    },
    showNextPokemon() {
      // Ein zufälliger Index, der sich von dem aktuellen unterscheidet
      let newIndex;
      do {
        newIndex = Math.floor(Math.random() * this.pokemons.length);
      } while (newIndex === this.currentLearningIndex);
      this.currentLearningIndex = newIndex;
      this.currentLearningPokemon = this.pokemons[this.currentLearningIndex];
    },
    scrollToPokemon(pokemonName) {
      const refName = `pokemon-${pokemonName}`;
      if (this.$refs[refName]) {
        const element = this.$refs[refName][0];
        element.scrollIntoView({ behavior: 'smooth' });
      } else {
        console.error('Keine Referenz gefunden für:', pokemonName);
      }
    },

    findClosestPhoneticPokemon(spokenWord) {
      const encodedSpokenWord = this.encoder.doubleMetaphone(spokenWord);
      let closestPokemon = null;
      let closestDistance = 3;

      for (let pokemon of this.pokemons) {
        const encodedPokemonName = this.encoder.doubleMetaphone(pokemon.name);
        // Vergleich der phonetischen Codes
        const distancePrimary = levenshtein.get(encodedSpokenWord.primary, encodedPokemonName.primary);
        const distanceAlternate = levenshtein.get(encodedSpokenWord.alternate, encodedPokemonName.alternate);
        const distance = Math.min(distancePrimary, distanceAlternate);

        if (distance < closestDistance) {
          closestDistance = distance;
          closestPokemon = pokemon;
        }
      }

      return closestPokemon;
    },
    
    scrollToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },

    getSpeechRecognizer() {
      const speechConfig = SpeechSDK.SpeechConfig.fromSubscription("901e7cb5bc09481abf6dd507019a3f35", "westeurope");
      const audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput();
      return new SpeechSDK.SpeechRecognizer(speechConfig, audioConfig);
    },

    
listenToPokemonName() {
  if (this.micButtonDisabled) {
    return;
  }
  this.isListening = true;
  this.micButtonDisabled = true;

  // Konfiguration für den SpeechRecognizer
  const speechConfig = SpeechSDK.SpeechConfig.fromSubscription("901e7cb5bc09481abf6dd507019a3f35", "westeurope");
  speechConfig.speechRecognitionLanguage = 'de-DE'; // Stellen Sie sicher, dass dies korrekt eingestellt ist

  const audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput();
  const recognizer = new SpeechSDK.SpeechRecognizer(speechConfig, audioConfig);

  // Erstellen der Phrase List
  const phraseList = SpeechSDK.PhraseListGrammar.fromRecognizer(recognizer);
  this.pokemons.forEach(pokemon => {
    phraseList.addPhrase(pokemon.name);
  });

  // Ausgabe der Phrase List zur Überprüfung
  console.log("Phrase List: ", this.pokemons.map(p => p.name).join(", "));

  recognizer.recognizeOnceAsync(result => {
    this.isListening = false;
    this.micButtonDisabled = false;

    if (result.reason === SpeechSDK.ResultReason.RecognizedSpeech) {
      const spokenWord = result.text.toLowerCase().trim();
      alert("Erkannter Text: " + spokenWord);

      const closestPokemon = this.findClosestPhoneticPokemon(spokenWord);
      this.recognizedText = closestPokemon.name;

      if (this.learningMode) {
        if (closestPokemon.name.toLowerCase() === this.currentLearningPokemon.name.toLowerCase()) {
          this.correctPokemonIndex = this.currentLearningIndex;
          this.successSound.play();
          this.correctPokemonIndex = null;  // Setzen Sie den Wert zurück, bevor das nächste Pokémon angezeigt wird
          this.showNextPokemon();
        } else {
          this.wrongSound.play();
        }
      } else {
        this.scrollToPokemon(closestPokemon.name);
      }
    } else {
      alert("Spracherkennung fehlgeschlagen: " + result.reason);
    }
  });
},

  }
};
</script>
