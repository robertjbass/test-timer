<template>
  <div class="timer">
    <div class="controls">
      <div class="settings-wrap">
        <div class="settings">
          <div class="btn">
            <button class="site" @click="editSettings = !editSettings">
              {{ editSettings ? '❌' : '✏️' }}
            </button>
          </div>
          <div class="input">
            <label
              :style="editSettings ? 'color: black' : 'color: lightgray'"
              for="numberOfQuestions"
              >Number of Questions:
            </label>
            <input
              :disabled="!editSettings"
              type="text"
              name="numberOfQuestions"
              v-model="numberOfQuestions"
            />
            <br />
            <label
              :style="editSettings ? 'color: black' : 'color: lightgray'"
              for="totalTimeInMins"
              >Test Length (Minutes):
            </label>
            <input
              :disabled="!editSettings"
              type="text"
              name="totalTimeInMins"
              v-model="totalTimeInMins"
            />
          </div>
        </div>
      </div>
      <section class="audio-area">
        <h4 style="color: black">Audio Settings</h4>
        <div
          :class="muted ? 'audio muted' : 'audio'"
          @click="playSpeech = !playSpeech"
        >
          {{ playSpeech ? '🗣' : '🛎' }} Audio
        </div>
        <div class="audio mute" @click="muted = !muted">
          {{ muted ? '⬛️ Mute' : '⬜️ Mute' }}
        </div>
      </section>
    </div>
    <div id="top" class="progress" style="display: flex;">
      <div
        class="completion-bar"
        style="width: 100%; background-color: black; position: absolute; top: 0; left: 0"
      ></div>
      <div
        class="completion-bar"
        :style="`width: ${completionBar}; position: absolute; top: 0; left: 0`"
      ></div>
    </div>
    <div id="bottom" class="progress" style="display: flex;">
      <div
        class="completion-bar"
        style="width: 100%; background-color: black; position: absolute; bottom: 0; left: 0"
      ></div>
      <div
        class="completion-bar"
        :style="
          `width: ${completionBar}; position: absolute; bottom: 0; left: 0`
        "
      ></div>
    </div>
    <h2>{{ secondsPerQuestion }} seconds per question<br /></h2>
    <div class="goals">
      <h3>{{ numberOfQuestions }} Questions</h3>
      <h4>Question Completion: {{ questionGoal }}</h4>
      <h4>Question Remaining: {{ numberOfQuestions - questionGoal }}</h4>
      <h3>Time {{ pad(tMins) }}:{{ pad(tSecs) }}</h3>
      <h4>Total Seconds Elapsed: {{ seconds }} / {{ timeLimit }}</h4>
      <h4>Total Minutes Elapsed: {{ minutes }} / {{ totalTimeInMins }}</h4>
      <h4>Percentage Complete: {{ completionBar }}</h4>
    </div>

    <button
      :class="!timerIsRunning ? 'button' : 'back'"
      @click="startButtonHandler"
      :style="`background-color: ${timerIsRunning ? 'blue' : 'red'}`"
    >
      {{ timerIsRunning ? 'Reset' : 'Start' }}
    </button>
    <hr />
    <a href="https://github.com/716green/test-timer">GitHub</a>
  </div>
</template>

<script>
  import next from '../assets/next.mp3';
  import ding from '../assets/ding.mp3';
  import half from '../assets/half.mp3';
  import threeQuarters from '../assets/threeQuarters.mp3';

  export default {
    name: 'Timer',
    data() {
      return {
        next,
        ding,
        half,
        threeQuarters,
        audio: null,
        muted: false,
        playSpeech: true,
        editSettings: false,
        timerIsRunning: false,
        seconds: 0,
        totalTimeInMins: 15,
        secondsElapsed: 0,
        numberOfQuestions: 50,
        questionGoal: 0,
        minutes: 0,
      };
    },
    watch: {
      seconds() {
        if (this.seconds % this.secondsPerQuestion === 0) {
          this.questionGoal++;
          this.playSound();
        }
        if (this.seconds % 60 === 0) {
          this.minutes++;
        }
        if (this.seconds === this.timeLimit / 2) {
          this.halfAlert();
        }
        if (this.seconds === (this.timeLimit / 4) * 3) {
          this.threeQuarterAlert();
        }
      },
      questionGoal() {
        if (this.questionGoal === this.numberOfQuestions) {
          this.timerIsRunning = false;
        }
      },
    },
    methods: {
      pad(value) {
        if (value.toString().length === 1) {
          return `0${value}`;
        } else {
          return value;
        }
      },
      reset() {
        document.location.reload();
      },
      halfAlert() {
        if (!this.muted) {
          const audio = new Audio(this.half);
          audio.play();
        }
      },
      threeQuarterAlert() {
        if (!this.muted) {
          const audio = new Audio(threeQuarters);
          audio.play();
        }
      },
      playSound() {
        if (this.playSpeech) {
          this.audio = new Audio(this.next);
        } else {
          this.audio = new Audio(this.ding);
        }
        this.audio.play();
      },
      runTimer() {
        setTimeout(() => {
          this.addSecond();
          if (this.seconds < this.timeLimit) {
            this.runTimer();
          }
        }, 1000);
      },
      startButtonHandler() {
        this.timerIsRunning = !this.timerIsRunning;
        if (this.timerIsRunning) {
          this.runTimer();
        } else {
          this.reset();
        }
      },
      addSecond() {
        if (this.timerIsRunning) {
          this.seconds++;
        }
      },
    },
    computed: {
      tMins() {
        return this.minutes;
      },
      tSecs() {
        if (this.seconds > 59) {
          return this.seconds % 60;
        } else return this.seconds;
      },
      secondsPerQuestion() {
        return this.timeLimit / this.numberOfQuestions;
      },
      timeLimit() {
        return this.totalTimeInMins * 60;
      },
      completionBar() {
        return ((this.seconds / this.timeLimit) * 100).toFixed(2) + '%';
      },
    },
  };
</script>

<style scoped>
  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }

  .completion-bar {
    background-color: slateblue;
    height: 30px;
    display: flex;
    margin: auto;
  }

  @import url(https://fonts.googleapis.com/css?family=Open+Sans:400);

  /* General Buttons */
  button {
    width: 130px;
    height: 40px;
    background: linear-gradient(to bottom, #4eb5e5 0%, #389ed5 100%); /* W3C */
    border: none;
    border-radius: 5px;
    position: relative;
    border-bottom: 4px solid #2b8bc6;
    color: #fbfbfb;
    font-weight: 600;
    font-family: 'Open Sans', sans-serif;
    text-shadow: 1px 1px 1px rgba(0, 0, 0, 0.4);
    font-size: 15px;
    text-align: left;
    text-indent: 5px;
    box-shadow: 0px 3px 0px 0px rgba(0, 0, 0, 0.2);
    cursor: pointer;

    /* Just for presentation */
    display: block;
    margin: 0 auto;
    margin-bottom: 20px;
  }
  button:active {
    box-shadow: 0px 2px 0px 0px rgba(0, 0, 0, 0.2);
    top: 1px;
  }

  button:after {
    content: '';
    width: 0;
    height: 0;
    display: block;
    border-top: 20px solid #187dbc;
    border-bottom: 20px solid #187dbc;
    border-left: 16px solid transparent;
    border-right: 20px solid #187dbc;
    position: absolute;
    opacity: 0.6;
    right: 0;
    top: 0;
    border-radius: 0 5px 5px 0;
  }

  /* Button pointing left */

  button.back {
    text-align: right;
    padding-right: 12px;
    box-sizing: border-box;
  }
  button.back:after {
    content: '';
    width: 0;
    height: 0;
    display: block;
    border-top: 20px solid #187dbc;
    border-bottom: 20px solid #187dbc;
    border-right: 16px solid transparent;
    border-left: 20px solid #187dbc;
    position: absolute;
    opacity: 0.6;
    left: 0;
    top: 0;
    border-radius: 5px 0 0 5px;
  }

  /* Single buttons */

  button.site {
    width: 40px;
    text-align: center;
    text-indent: 0;
  }
  button.site:after {
    display: none;
  }

  /* Presentation stuff */
  .holder {
    width: 400px;
    background: #efefef;
    padding: 30px 10px;
    box-sizing: border-box;
    margin: 0 auto;
    margin-top: 20px;
    text-align: center;
  }

  h1 {
    font: 400 16px 'Open Sans';
    text-transform: uppercase;
    color: #999;
    text-shadow: 1px 1px 1px #fff;
    margin-bottom: 30px;
  }

  input {
    width: 30px;
  }

  .settings-wrap {
    justify-content: center;
    width: 100%;
    display: flex;
    margin: auto;
    font-weight: 500;
  }

  .settings {
    display: flex;
    height: 50px;
  }

  .btn {
    margin-right: 20px;
  }

  .audio {
    cursor: pointer;
  }

  .audio:hover {
    color: lightcoral;
  }

  .muted {
    text-decoration: line-through;
  }

  .audio-area {
    color: purple;
    font-weight: 500;
  }
</style>
