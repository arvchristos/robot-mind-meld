<template>
  <main id="app">
    <a href="https://github.com/alex-lew/robot-mind-meld"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://camo.githubusercontent.com/e7bbb0521b397edbd5fe43e7f760759336b5e05f/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f677265656e5f3030373230302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_green_007200.png"></a>
    <img id="logo" alt="Me, a friendly robot!" src="./assets/bot.png">
    <h1>Robot Mind Meld</h1>
    <div id="bot-display">
      <div id="headline">
        <span v-if="round == 0">Hello, human person! Wanna play <a href="https://wiki.improvresourcecenter.com/index.php?title=Mind_Meld">an improv game</a>?</span>
        <span v-else-if="!finished">Attempt #{{round}} at Connection with Human Person</span>
        <span v-else>Attempt #{{round}} — Connected with Human Person!</span>
      </div>
      <div id="conversation">
        <div class="dialogue-box">
          <img class="emojicon" src="./assets/me.png" alt="a robot emoji"  :title="nextRobotWord">
          <transition mode="out-in" name="slide-fade">
            <p class="caption" :key="currentRobotWord">
              {{currentRobotWord}}
            </p>
          </transition>
        </div>
        <div class="dialogue-box">
          <img class="emojicon" src="./assets/you.png" alt="a human emoji">
          <transition mode="out-in" name="slide-fade">
            <p class="caption" :key="currentHumanWord">
              {{currentHumanWord}}
            </p>
          </transition>
        </div>
      </div>
    </div>
    <transition mode="out-in" name="fade">
      <div v-if="!finished" id="playing" key="playing">
        <div id="instructions">
          <transition mode="out-in" name="fade">
            <p v-if="round == 0" :key="round">
              To get us started, think of <em>any</em> word.<br>I will also think of any word. We’ll say them at the same time!
            </p>
            <p v-else-if="round == 1" :key="round">
              The goal is to achieve <em>mind meld</em>: to say the <em>same word simultaneously</em>.
              <br>This first attempt failed, but don’t despair! Let’s try again: think of a word
              related to <em>both</em> <span class="highlight">{{currentHumanWord}}</span> <em>and</em> <span class="highlight">{{currentRobotWord}}</span>. 
              I will too.
              Fingers crossed! 🤞
            </p>
            <p v-else :key="round">
              <span v-if="currentCloseness < 0.05">Odd combination... This should be fun!</span>
              <span v-else-if="currentCloseness > 0.25">Ahh, so close! Let’s try again.</span>
              <span v-else>We’ll get there! Let’s try again.</span>
              <br>
              <span v-if="currentCloseness < 0.05">Think we can find something related to <em>both</em> <span class="highlight">{{currentHumanWord}}</span> <em>and</em> <span class="highlight">{{currentRobotWord}}</span>?</span>
              <span v-else>Something related to <em>both</em> <span class="highlight">{{currentHumanWord}}</span> <em>and</em> <span class="highlight">{{currentRobotWord}}</span>.</span>
            </p>
          </transition>
        </div>
        <div id="input-form">
          <transition name="fade-error">
            <p id="error-message" v-if="!nextWordValid && nextHumanWord == ''">
              I'm sorry, I don’t know that word! Try another?
            </p>
          </transition>
          <input :disabled="waiting" ref="textbox"  v-on:keyup.enter="sayWord" v-model="nextHumanWord" autocapitalize="none"  :class="{error: !nextWordValid && nextHumanWord==''}" :placeholder="round == 0 ? 'type it here...' : ''">
          <button :disabled="nextHumanWord == ''" v-on:click="sayWord">say it now!</button>
          <div id="give-up-buttons">
            <a id="start-over" v-if="round != 0" href="/">start over</a>
            <a id="close-enough" v-if="currentCloseness > 0.75" v-on:click="winPrematurely">these words are already the same, silly robot</a>
          </div>
        </div>
        <p id="acknowledgments">I was designed by <a href="http://alexlew.net">Alex Lew</a>, and am powered by <a href="https://blog.conceptnet.io/2016/05/25/conceptnet-numberbatch-a-new-name-for-the-best-word-embeddings-you-can-download/">Conceptnet Numberbatch</a>, a set of “word embeddings” that allow me to think quantitatively about words and their relationships. My face (and yours!) come from <a href="https://www.emojione.com/">EmojiOne</a>.</p>
      </div>
      <div v-else id="gameover" key="gameover">
        <p id="victory-banner">WE DID IT!</p>
        <p>Our minds have melded! Let’s remember our journey to connection:</p>
        <div id="summary">
          <table>
            <tr><th>Round</th><th>Me</th><th>You</th></tr>
            <tr v-for="(round, index) in guessedWords" :key="round">
              <td>{{index+1}}</td><td>{{round[0]}}</td><td>{{round[1]}}</td>
            </tr>
          </table>
        </div>
        <div id="about">
          <p v-if="round < 10">That was fast! And I promise, I didn’t cheat: like a human, I decide on each word I say <em>before</em> you reveal what you’re thinking. (Don’t believe me? Whenever you’d like, you can test me. Just <span v-if="isTouchScreen">rest your finger lightly on</span> <span v-else>hover your mouse over</span> my face, and I’ll reveal the word I’ve chosen!)</p>
          <p>I was designed by <a href="http://alexlew.net">Alex Lew</a>, and am powered by <a href="https://blog.conceptnet.io/2016/05/25/conceptnet-numberbatch-a-new-name-for-the-best-word-embeddings-you-can-download/">Conceptnet Numberbatch</a>, a set of “word embeddings” that allow me to think quantitatively about words and their relationships. Constructed by analyzing millions of documents for patterns, these “numberbatches” now play a role in most programs that process language, from speech recognition to machine translation.</p> 
          <p>This game is meant to serve as a fun exploration of the ways that machines are beginning to understand our language—and, by extension, our world.</p> 
          <a href="/">Play again?</a>
          </div>
      </div>
    </transition>
  </main>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      currentHumanWord: 'this is you',
      currentRobotWord: 'this is me',
      guessedWords: [],
      nextHumanWord: '',
      nextRobotWord: '',
      round: 0,
      currentCloseness: 0,
      nextWordValid: true,
      waiting: true,
      finished: false,
      isTouchScreen: 'ontouchstart' in window
    }
  },
  mounted: function () {
    fetch('/first_word').then(response => response.json()).then(({word}) => {
      this.nextRobotWord = word
      this.waiting = false
    })
  },
  methods: {
    winPrematurely: function () {
      this.finished = true
    },
    sayWord: function () {
      if (this.waiting || this.nextHumanWord === '') { return }

      // Clear the text field to disable the button,
      // but save its contents
      this.waiting = true
      const nextHumanWord = this.nextHumanWord
      this.nextHumanWord = ''
      this.nextWordValid = true

      // Make the request
      fetch('/next_word', {
        method: 'POST',
        headers: {
          'content-type': 'application/json'
        },
        body: JSON.stringify({
          past: [].concat(...this.guessedWords),
          word1: nextHumanWord,
          word2: this.nextRobotWord
        })
      })
        .then(function (response) {
          if (!response.ok) {
            this.waiting = false
          }
          return response.json()
        })
        .then((resp) => {
          this.waiting = false
          this.$nextTick(() => this.$refs.textbox.focus())
          this.nextWordValid = !resp.unknownWord
          if (!this.nextWordValid) { return }

          // resp.victory indicates that the human and robot words basically matched,
          // up to stemming; we display the human word for both in this case, to make
          // it seem just a tiny bit more magical
          this.currentRobotWord = resp.victory ? nextHumanWord.toLowerCase() : this.nextRobotWord
          this.currentHumanWord = nextHumanWord
          this.guessedWords.push([this.currentRobotWord, this.currentHumanWord])
          this.round += 1
          this.finished = resp.victory
          this.currentCloseness = resp.simScore
          this.nextRobotWord = resp.nextWord
        }).catch(_ => this.waiting = false)
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", "Charter", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding-top: 60px;
  display: flex;
  flex-direction: column;
  padding-bottom: 30px;
  margin: auto;
}

#app div {
  margin: auto;
}

#app h1 {
  font-weight: 600;
}

#app a {
  color: rgba(60, 150, 240, 0.85);
}

#app #logo {
  width: 200px;
  margin: auto;
}

@media screen and (max-width: 350px) {
  #app #logo {
    width: 150px;
  }
}

/* BOT DISPLAY BOX */

#bot-display {
  /* TODO: Use media queries to create mobile-friendly version */
  width: 450px;
  border: solid 1px #aaa;
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
  border-radius: 5px;
}

#bot-display #headline {
  padding: 10px;
  color: deeppink;
  font-weight: 500;
  border-bottom: solid 1px #aaa;
  margin: 0;
}

#bot-display #conversation {
  display: flex;
  flex-direction: row;
  margin: 0;
}

#bot-display #conversation .dialogue-box {
  flex-basis: 0;
  flex-grow: 1;
}

#bot-display #conversation .dialogue-box:first-of-type {
  border-right: solid 1px #aaa;
}

#bot-display #conversation .dialogue-box .emojicon {
  width: 75px;
  margin-top: 30px;
  line-height: 0;
}

#bot-display #conversation .dialogue-box .caption {
  font-size: 1.5em;
  margin-top: 5px;
}

/* PLAYING */

#playing {
  display: flex;
  flex-direction: column;
}

#playing #instructions {
  line-height: 1.5em;
  font-size: 1.1em;
  font-weight: 500;
  max-width: 600px;
  margin-top: 10px;
}

#playing #instructions .highlight {
  color: rgba(60, 150, 240, 0.85);
  font-weight: 600;
}

#playing #input-form {
  display: flex;
  flex-direction: column;
  width: 400px;
}

#playing #input-form input {
  background: #eee;
  font-size: 1.5em;
  padding: 15px;
  text-align: center;
  border: none;
  margin-bottom: 5px;
  border-radius: 5px;
}

#playing #input-form input.error {
  border: solid 1px red;
}

#playing #input-form #error-message {
  color: red;
  margin-bottom: 0;
  margin-top: 0;
  display: block;
}


#playing #give-up-buttons {
  display: flex;
  margin: auto;
}

#playing #input-form #start-over, #playing #input-form #close-enough {
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1em;
  text-decoration: none;
  padding: 3px;
  border: none;
  border-radius: 5px;
  margin: 20px 5px;
  width: 100px;
}

#start-over {
  background: rgba(255, 28, 20, 0.8);
}

#playing #input-form #close-enough {
  background: rgba(60, 150, 240, 0.85);
  width: 200px;
  cursor: pointer;
}


#playing #input-form #start-over:hover {
  background: rgba(255, 28, 20, 0.5);
}

#playing #input-form #close-enough:hover {
  background: rgb(71, 180, 253);
}


#playing #input-form button {
  background: rgb(71, 180, 253);
  color: white;
  cursor: pointer;
  font-size: 1.5em;
  padding: 10px;
  border: none;
  border-radius: 5px;
}

#playing #input-form button:disabled,
#playing #input-form button:disabled:hover {
  background: rgba(71, 180, 253, 0.5);
  cursor: default;
}

#playing #input-form button:hover {
  background: rgba(90, 199, 253, 0.85);
}

#playing #input-form button:active {
  background: rgba(60, 150, 240, 0.85);
}

#playing #acknowledgments {
  max-width: 550px;
  margin-top: 30px;
  color: gray;
  border-top: dashed 1px gray;
  font-size: 0.8em;
  padding-top: 10px;
}

/* GAME OVER */

#gameover {
  display: flex;
  flex-direction: column;
}

#gameover #victory-banner {
  animation: pulsing 2s infinite;
  font-weight: 600;
  font-size: 3em;
  line-height: .5em;
  margin-top: 1em;
  margin-bottom: .5em;
}

#gameover > p {
  line-height: 1.5em;
  font-weight: 600;
  margin-top: 0;
  margin-bottom: 10px;
}

#gameover #summary {
  margin: auto;
  font-weight: normal;
  border: solid 1px #aaa;
}

#gameover #summary table {
  border-collapse: collapse;
}

#gameover #summary table td, #gameover #summary table th {
  padding-left: 10px;
  padding-right: 10px;
}

#gameover #summary table th {
  padding-top: 5px;
  padding-bottom: 5px;
}

#gameover #summary table tr:first-of-type {
  border-bottom: solid 1px #aaa;
}

#gameover #about {
  font-weight: 500;
  line-height: 1.5em;
  max-width: 500px;
}

/* ANIMATION */

@keyframes pulsing {
  0% {
    color: rgba(255, 20, 147, .2);
  }
  50% {
    color: deeppink;
  }
  100% {
    color: rgba(255, 20, 147, .2);
  }
}

.fade-enter-active,
.fade-leave-active,
.fade-error-enter-active {
  transition: opacity 1s;
}

.fade-enter,
.fade-leave-to,
.fade-error-enter,
.fade-error-leave-to {
  opacity: 0;
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}

.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter,
.slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}


/* 

Sizes:

maximum is 450, which you see at 475 and above.
between 450 and 475, you see 425
between 425 and 450, you see 400
between 375 and 400

*/


@media screen and (max-width: 450px) {
  #app, #bot-display, #playing #input-form, #playing #acknowledgments {
    width: 400px;
  }
}

@media screen and (max-width: 425px) {
  #app, #bot-display, #playing #input-form, #playing #acknowledgments {
    width: 375px;
  }
}

@media screen and (max-width: 400px) {
  #app, #bot-display, #playing #input-form, #playing #acknowledgments {
    width: 350px;
  }
}

@media screen and (max-width: 375px) {
  #app, #bot-display, #playing #input-form, #playing #acknowledgments {
    width: 325px;
  }
  #playing #input-form #close-enough {
    width: 175px;
  }
}


@media screen and (max-width: 350px) {
  #app, #bot-display, #playing #input-form, #playing #acknowledgments {
    width: 300px;
  }
  #playing #input-form #close-enough {
    width: 150px;
  }
}

@media screen and (max-height: 500px) {
  #app #logo {
    width: 90px;
  }
}
</style>
