<template>
  <div class="typing-test-inscript-hindi">
    <section class="hero is-dark" id="typing-content" v-once>
      <div class="hero-body" style="padding: 2rem 1.5rem;">
        <div class="container">
          <h1 class="title" id="lesson" style="line-height: 1.6; font-size: 27px;"></h1>
        </div>
        <div id="action-new-lesson" style="display: none;">
          <nav class="level">
            <div class="level-item has-text-centered">
              <!-- <b-tooltip label="Previous Lesson" type="is-warning" animated always>
                <button class="button is-medium is-warning">
                  <b-icon size="is-large" icon="skip-previous"></b-icon>
                </button>
              </b-tooltip>-->
            </div>
            <div class="level-item has-text-centered">
              <b-tooltip label="Play Again" type="is-success" animated>
                <button @click="playAgainTyping" class="button is-medium is-success">
                  <b-icon size="is-medium" icon="play"></b-icon>
                </button>
              </b-tooltip>&nbsp; &nbsp; &nbsp;
              <b-tooltip label="Reload Passage" type="is-primary" animated>
                <button @click="reloadTypingPassage" class="button is-medium is-primary">
                  <b-icon size="is-medium" icon="reload"></b-icon>
                </button>
              </b-tooltip>
            </div>
            <div class="level-item has-text-centered">
              <!-- <b-tooltip label="Next Lesson" type="is-warning" animated always>
                <button class="button is-medium is-warning">
                  <b-icon size="is-large" icon="skip-next"></b-icon>
                </button>
              </b-tooltip>-->
            </div>
          </nav>
        </div>
      </div>
    </section>
    <!-- Result Section -->
    <section id="resultShow" class="hero is-warning">
      <div class="hero-body" style="padding: 1rem 0.5rem;">
        <nav class="level">
          <div class="level-item has-text-centered">
            <div title="Gross Word Per Minute">
              <p class="heading">GWPM</p>
              <p class="title">{{ result.GrossWPM }}</p>
            </div>
          </div>
          <div class="level-item has-text-centered">
            <div title="Net Word Per Minute">
              <p class="heading">NWPM</p>
              <p class="title">{{ result.NetWPM }}</p>
            </div>
          </div>
          <div class="level-item has-text-centered">
            <div title="Accuracy">
              <p class="heading">ACCURACY ({{ result.totalWrongWords }})</p>
              <p class="title">{{ result.Accuracy }} %</p>
            </div>
          </div>
          <div class="level-item has-text-centered">
            <div title="Total Character">
              <p class="heading">Total Character</p>
              <p class="title">{{ result.totalChar }}</p>
            </div>
          </div>
          <div class="level-item has-text-centered">
            <div title="Total Words">
              <p class="heading">Total Words ({{ result.totalWords }})</p>
              <p class="title">{{ result.typingWordIndex }}</p>
            </div>
          </div>
          <!-- Countdown -->
          <div class="level-item has-text-centered">
            <div>
              <p class="heading">TIME</p>
              <p class="title" title="Countdown" v-if="result.totalTimeDuration !== 'free'">
                <countdown
                  :key="countDownKey"
                  :time="result.totalTimeDuration"
                  :auto-start="false"
                  ref="countdown"
                  @countdownprogress="handleCountdownProgress"
                  @countdownend="handleCountdownEnd"
                >
                  <template slot-scope="props">{{ props.minutes }}:{{ props.seconds }}</template>
                </countdown>
              </p>
              <p class="title" v-else title="Timer">
                <b-tooltip
                  label="Click to Pause Typing"
                  position="is-bottom"
                  type="is-danger"
                  animated
                  always
                >
                  <span @click="startTimer('stop')">{{ timerSettings.Output }}</span>
                </b-tooltip>
              </p>
            </div>
          </div>
          <!-- Settings -->
          <b-tooltip label="Settings" position="is-left" type="is-dark" animated>
            <a class="button" @click="isSettingsModalActive = true">
              <b-icon icon="settings"></b-icon>
            </a>
          </b-tooltip>
        </nav>
      </div>
    </section>
    <!-- Show Word Hint -->
    <div class="container" v-show="isShowWordHint">
      <div style="font-size: xx-large; padding-top: 17px;">
        <b-tooltip label="Word Hint" type="is-info" animated>
          <b-tag type="is-info" size="is-medium">
            <b-icon icon="lightbulb-on"></b-icon>
          </b-tag>
        </b-tooltip>&nbsp;
        <strong style="letter-spacing: 8px;" v-for="w in hintWord" :key="w.index">
          <span v-if="w.isUnicodeKey">
            <b-tooltip
              :label="'Alt + '+ w.altUnicodeKey"
              style="letter-spacing: 0px; font-family: cursive;"
              always
            >
              <span
                class="tag is-danger is-large"
                style="margin-right: 8px; padding-left: 7px; padding-right: 7px;"
              >{{ w.string }}</span>
            </b-tooltip>
          </span>
          <span v-else>{{ w.string }}</span>
        </strong>
      </div>
    </div>
    <!-- Typing Section -->
    <section class="section" v-once>
      <div class="container">
        <div class="field">
          <div class="control">
            <input
              class="input is-primary is-large"
              id="typingBox"
              @keydown="keyDownEventFunction"
              type="text"
              placeholder="Start Typing.."
              autocomplete="off"
            >
          </div>
        </div>
      </div>
    </section>
    <!-- Settings Model -->
    <b-modal :active.sync="isSettingsModalActive" :width="640" scroll="keep">
      <div class="card">
        <header class="card-header has-background-grey-light">
          <p class="card-header-title">Typing Settings</p>
        </header>
        <div class="card-content">
          <section>
            <div class="field">
              <b-switch v-model="isShowOnlyOneWord">Only Show Single Word in Typing Box</b-switch>
            </div>
            <div class="field">
              <b-switch v-model="isCheckOneByOneChar">Check One-by-One Character in Word</b-switch>
            </div>
          </section>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script>
import Vue from "vue";

import VueCountdown from "@xkeshi/vue-countdown";
Vue.component(VueCountdown.name, VueCountdown);

import { Base64 } from "js-base64";

import MX_Typing_Test from "../MX_Typing_Test.js";

export default {
  name: "typing-test-inscript-hindi",
  mixins: [MX_Typing_Test],

  data() {
    return {
      hintWord: false,
      typingContentDivClass: "inscriptHindiWords" // This Class name Using in CSS
    };
  },

  created() {
    var lid = Base64.decode(this.$route.params.id);
    var PType = this.$route.params.passageType;
    if (this.$store.getters["iht/hasLesson"](lid, PType)) {
      this.$Progress.start();
      this.$store.getters["iht/getLesson"](lid, PType).then(module => {
        // User Choise Typing Words Length
        var userSelectWordsLength = this.$store.state.iht
          .selectedTypingWordsLength;
        var passageWordsLength = this.countWords(module.default.passage);

        // console.log("User Select Words Length: " + userSelectWordsLength);
        // console.log("Passage Words Length: " + passageWordsLength);

        if (PType === "beginner") {
          // Shuffle Passage
          if (userSelectWordsLength <= passageWordsLength) {
            if (userSelectWordsLength === passageWordsLength) {
              var passageShuffle = this.shuffle((module.default.passage).split(" "))
              this.typingContent = passageShuffle.join(" ")
            } else {
              var passageShuffle = this.shuffle(module.default.passage.split(" ").slice(0, -(passageWordsLength - userSelectWordsLength)))
              this.typingContent = passageShuffle.join(" ")
            }
          } else {
            // Passage Words Length: Less (??????)
            var passMulti = Math.ceil(userSelectWordsLength / passageWordsLength); // passageWordsLength * passMulti
            var newPassage = (module.default.passage + " ").repeat(passMulti);

            var passageShuffle = this.shuffle(newPassage.split(" ").slice(0, -(this.countWords(newPassage) - userSelectWordsLength)))
            this.typingContent = passageShuffle.join(" ")
          }
        } else {

          if (userSelectWordsLength <= passageWordsLength) {
            if (userSelectWordsLength === passageWordsLength) {
              this.typingContent = module.default.passage;
            } else {
              this.typingContent = module.default.passage
                .split(" ")
                .slice(0, -(passageWordsLength - userSelectWordsLength))
                .join(" ");
            }
          } else {
            // Passage Words Length: Less (??????)
            var passMulti = Math.ceil(userSelectWordsLength / passageWordsLength); // passageWordsLength * passMulti
            var newPassage = (module.default.passage + " ").repeat(passMulti);
  
            this.typingContent = newPassage
              .split(" ")
              .slice(0, -(this.countWords(newPassage) - userSelectWordsLength))
              .join(" ");
          }
        }

        this.$Progress.finish();
        this.generateWords();
      });
    }
  },

  computed: {
    timeDuration() {
      return this.$store.state.iht.selectedTimeDuration;
    },
    isDisableBackSpace() {
      return this.$store.state.iht.isDisableBackSpace;
    },
    isShowWordHint() {
      return this.$store.state.iht.isShowWordHint;
    }
  },

  methods: {
    /**
     * Reload Typing Passage
     */
    reloadTypingPassage() {
      var PType = this.$route.params.passageType;

      // beginner, normal, advanced
      if (PType === "beginner") {
        var tPassage = this.$store.state.iht.listBeginnerPassages.length;
        var gNum = Math.floor(Math.random() * (tPassage - 1 + 1)) + 0;
        var lessonID = this.$store.state.iht.listBeginnerPassages[gNum].id;
        this.$router.push({
          name: "typing-test-inscript-hindi",
          params: { passageType: "beginner", id: Base64.encode(lessonID) }
        });
      } else if (PType === "normal") {
        var tPassage = this.$store.state.iht.listNormalPassages.length;
        var gNum = Math.floor(Math.random() * (tPassage - 1 + 1)) + 0;
        var lessonID = this.$store.state.iht.listNormalPassages[gNum].id;
        this.$router.push({
          name: "typing-test-inscript-hindi",
          params: { passageType: "normal", id: Base64.encode(lessonID) }
        });
      } else {
        var tPassage = this.$store.state.iht.listAdvancedPassages.length;
        var gNum = Math.floor(Math.random() * (tPassage - 1 + 1)) + 0;
        var lessonID = this.$store.state.iht.listAdvancedPassages[gNum].id;
        this.$router.push({
          name: "typing-test-inscript-hindi",
          params: { passageType: "advanced", id: Base64.encode(lessonID) }
        });
      }
    },

    /**
     * Find Unicode Character in String
     * @return {array}
     */
    findUnicodeCharacter(str) {
      var returnData = [];
      for (var i = 0, n = str.length; i < n; i++) {
        var out = this.$store.getters["findUChar/findInscriptAltKey"](str[i]);
        if (out === false) {
          returnData.push({
            string: str[i],
            isUnicodeKey: false,
            altUnicodeKey: false
          });
        } else {
          returnData.push({
            string: str[i],
            isUnicodeKey: true,
            altUnicodeKey: out
          });
        }
      }
      return returnData;
    },

    /**
     * Shuffle Array
     * @param: {array}
     */
    shuffle(array) {
      var currentIndex = array.length,
        temporaryValue,
        randomIndex;

      // While there remain elements to shuffle...
      while (0 !== currentIndex) {
        // Pick a remaining element...
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;

        // And swap it with the current element.
        temporaryValue = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temporaryValue;
      }

      return array;
    }
  },

  watch: {
    currentTypingWord(v) {
      if (this.isShowWordHint === true) {
        if (v !== undefined) {
          this.hintWord = this.findUnicodeCharacter(v);
        }
      }
    }
  }
};
</script>

<style>
/* Overlay */
#action-new-lesson {
  height: 50%;
  position: absolute;
  left: 0%;
  right: 0%;
  top: 0%;
  bottom: 13%;
  margin: auto;
}

/** Typing Content Div */
#inscriptHindiWords {
  width: 96.4%;
  height: 285px;
  overflow-y: scroll;
}

/**
 * Custom Scrollbar Styling
 * https://codepen.io/devstreak/pen/dMYgeO
 */
#inscriptHindiWords::-webkit-scrollbar-track {
  /* -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3); */
  /* background-color: #f5f5f5; */
  border-radius: 10px;
}

#inscriptHindiWords::-webkit-scrollbar {
  width: 10px;
}

#inscriptHindiWords::-webkit-scrollbar-thumb {
  background-image: -webkit-gradient(
    linear,
    left bottom,
    left top,
    color-stop(0.44, rgb(122, 153, 217)),
    color-stop(0.72, rgb(73, 125, 189)),
    color-stop(0.86, rgb(28, 58, 148))
  );
}
</style>
