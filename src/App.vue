<template>
  <div class="simon">
    <h1 class="simon__title">Simon</h1>
    <div class="simon__inner">
      <div class="simon__sections" ref="sections">
        <div class="simon__section" ref="section1" data-section="1"></div>
        <div class="simon__section" ref="section2" data-section="2"></div>
        <div class="simon__section" ref="section3" data-section="3"></div>
        <div class="simon__section" ref="section4" data-section="4"></div>
      </div>
      <div class="simon__settings">
        <div class="simon__difficulty">
          <h3 class="simon__difficulty-title">
            Уровень сложности
          </h3>
          <DifficultySelect
            v-model="currentDifficultyLevel"
            :difficultyLevels="difficultyLevels"
            :disabled="difficultySelectDisabled"
          />
        </div>
        <div class="simon__score">Счёт: {{ score }}</div>
        <div class="simon__settings-bottom">
          <button :disabled="startButtonDisabled" type="button" @click="playNextLevel">Начать</button>
          <div class="simon__game-over" v-if="isGameOver">Игра окончена со счётом {{ finalScore }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import DifficultySelect from "@/components/DifficultySelect.vue";

export default {
  components: { DifficultySelect },
  data() {
    return {
      currentSections: [],
      currentUserSections: [],
      sounds: [],
      score: 0,
      finalScore: 0,
      isGameOver: false,
      startButtonDisabled: false,
      currentDifficultyLevel: "easy",
      difficultySelectDisabled: false,
      difficultyLevels: {
        easy: {
          text: "Лёгкий",
          time: 1500,
        },
        medium: {
          text: "Средний",
          time: 1000,
        },
        hard: {
          text: "Сложный",
          time: 400,
        },
      },
    }
  },
  methods: {
    getRandomSection() {
      return Math.floor(Math.random() * 4 + 1);
    },
    playNextLevel() {
      this.startButtonDisabled = true;
      this.difficultySelectDisabled = true;
      
      if (this.currentSections.length === 0) {
        this.isGameOver = false;
      }
      
      let count = 0;
      // убираем обработчик клика, чтобы нельзя было нажимать на секции, пока воспроизводится порядок
      this.removeSectionsListener();
      this.currentSections.push(this.getRandomSection());
      
      let interval = setInterval(() => {
        this.$refs["section" + this.currentSections[count]].style.opacity = "0.5";
        this.playSound(this.currentSections[count]);
        
        // таймаут для подсветки секций
        setTimeout(() => {
          this.$refs["section" + this.currentSections[count]].style = "";
          count++;
          
          // если прошлись по всем секциям в массиве, то убираем интервал и добавляем прослушку кликов
          if (count === (this.currentSections.length)) {
            clearInterval(interval);
            interval = null;
            this.addSectionsListener();
          }
        }, 300);
      }, this.difficultyLevels[this.currentDifficultyLevel].time);
    },
    addSectionsListener() {
      this.$refs.sections.addEventListener("mousedown", this.userClickListener);
    },
    removeSectionsListener() {
      this.$refs.sections.removeEventListener("mousedown", this.userClickListener);
    },
    userClickListener(e) {
      if (e.currentTarget !== e.target) {
        const sectionId = +e.target.dataset.section;
        this.currentUserSections.push(sectionId);
        this.playSound(sectionId);

        // проверяем, правильно ли юзер воспроизводит текущую последовательность секций
        const userIsRight = this.currentUserSections.every((section, index) => section === this.currentSections[index]);

        if (userIsRight) {
          if (this.currentSections.length === this.currentUserSections.length) {
            this.removeSectionsListener();
            this.currentUserSections = [];
            this.score++;
            this.playNextLevel();
          }
        } else {
          this.removeSectionsListener();
          this.finalScore = this.score;
          this.score = 0;
          this.currentSections = [];
          this.currentUserSections = [];
          this.isGameOver = true;
          this.startButtonDisabled = false;
          this.difficultySelectDisabled = false;
        }
      }
    },
    playSound(sectionId) {
      /*
      останавливаю воспроизведение всех звуков перед воспроизведением текущего, т.к. если быстро нажимать на секции,
      то звуки перекрывают друг друга и не успевают воспроизводиться 
      */
      this.sounds.forEach(sound => {
        sound.pause();
        sound.currentTime = 0.0;
      });
      
      this.sounds[sectionId - 1].play();
    },
  },
  mounted() {
    this.sounds = [
      new Audio("https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"),
      new Audio("https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"),
      new Audio("https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"),
      new Audio("https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"),
    ];
  },
}
</script>

<style scoped>
.simon {
    @media (max-width: 460px) {
        padding: 0 10px;
    }
}

.simon__title {
    text-align: center;
    margin-bottom: 50px;

    @media (max-width: 767px) {
        margin-bottom: 30px;
    }
}

.simon__inner {
    display: flex;
    justify-content: center;

    @media (max-width: 767px) {
        flex-direction: column-reverse;
        width: max-content;
        margin: 0 auto;
    }

    @media (max-width: 460px) {
        width: 100%;
    }
}

.simon__sections {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-gap: 20px;
    margin-right: 50px;
    width: 420px;
    aspect-ratio: 1/1;
    
    @media (max-width: 767px) {
        margin-right: 0;
    }

    @media (max-width: 460px) {
        width: 100%;
    }
}

.simon__section {
    cursor: pointer;
}

.simon__section:hover {
    opacity: 0.5;
}

.simon__section:nth-child(1) {
    background-color: deepskyblue;
}

.simon__section:nth-child(2) {
    background-color: red;
}

.simon__section:nth-child(3) {
    background-color: lawngreen;
}

.simon__section:nth-child(4) {
    background-color: yellow;
}

.simon__settings {
    min-width: 185px;

    @media (max-width: 767px) {
        margin-bottom: 30px;
    }
}

.simon__settings-bottom {
    display: flex;
    flex-direction: column;
    
    @media (max-width: 767px) {
        flex-direction: row;
        justify-content: space-between;
    }
}

.simon__difficulty {
    margin-bottom: 20px;
}

.simon__difficulty-title {
    font-size: 18px;
}

.simon__score {
    font-weight: 700;
    font-size: 18px;
    margin-bottom: 20px;
}

.simon__game-over {
    margin-top: 20px;

    @media (max-width: 767px) {
        margin-top: 0;
    }
}
</style>
