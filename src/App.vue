<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <div class="container">
    <div style="display: flex; flex-direction: row">
      <div
        class="button"
        @click="clickOnPost"
        v-bind:class="{ onHover: postIsOnHover }"
      >
        ✍️
      </div>
      <!-- <div class="button" @click="clickOnRefresh"
          v-bind:class="{onHover: refreshIsOnHover}"
          >
            🔄
          </div> -->
      <div
        class="button"
        @click="clickOnTrash"
        v-bind:class="{ onHover: onTrashPage }"
      >
        🗑️
      </div>
    </div>

    <transition name="open-input">
      <div class="inputBox" v-if="inputBoxShown">
        <div class="description">choose your anonymous identity</div>
        <div class="description">
          <select v-model="identity">
            <option
              v-for="option in options"
              v-bind:value="option.value"
              v-bind:key="option.value"
            >
              {{ option.value }}
            </option>
          </select>
        </div>
        <div class="rc-infoCard">
          <textarea
            name="textarea"
            placeholder="Type something here..."
            v-model="inputContent"
          ></textarea>
        </div>
        <div
          class="button"
          @click="clickOnSubmit"
          v-bind:class="{ onHover: submitIsOnHover }"
        >
          ✅
        </div>
      </div>
    </transition>

    <div v-if="onTrashPage" class="description">
      Deleted messages will be kept for only 24 hours.
    </div>

    <div v-if="msgList.length == 0 && !onTrashPage" class="description">
      Oops, someone cleared them out. Be the first one to ✍️.
    </div>

    <!-- <div style="font-size: 15px;text-align: center;margin: 10px;color:rgb(129, 129, 129);">
          Click on the delete button to remove one message
        </div> -->

    <div v-for="(item, index) in msgList" :key="index">
      <div class="rc-infoCard">
        <div class="delete">
          <div
            v-if="!onTrashPage"
            @click="clickOnDelete"
            :id="getForId(item._id)"
            :data-index="index"
          >
            ❌
          </div>
          <div v-else style="height: 20px"></div>
        </div>
        <div class="rc-infoDetail">
          {{ item.msg }}
          <div class="time" v-if="onTrashPage">
            ❌: {{ momentAgo(item.deleteTime) }}
          </div>
          <div class="time">
            <span v-if="onTrashPage">
              ✍️:
              <!-- {{this.momentAgo(item.time)}} -->
            </span>
            {{ momentAgo(item.time) }}
          </div>
        </div>
      </div>
    </div>

    <div class="divLine"></div>
    <a href="https://www.afdian.net/@hi_keon">
      <img
        id="banner"
        src="https://i.loli.net/2020/07/11/iUh6QFTXMormBnH.png"
        alt=""
      />
    </a>
  </div>
</template>

<script>
import "@/assets/roundCorner.css";
import "amfe-flexible";
import axios from 'axios';
import moment from 'moment';
const HOSTNAME = "https://thewallengine.gallinula.com";

export default {
  name: "App",
  components: {},
  data() {
    return {
      identity: "",
      options: [
        { value: "😀" },
        { value: "🤡" },
        { value: "😈" },
        { value: "🥶" },
        { value: "👾" },
        { value: "👻" },
        { value: "😼" },
        { value: "💩" },
        { value: "🐼" },
        { value: "🌚" },
        { value: "🍉" },
        { value: "🍔" },
        { value: "🏀" },
        { value: "🔥" },
      ],
      postIsOnHover: false,
      submitIsOnHover: false,
      inputBoxShown: false,
      // refreshIsOnHover: false,
      trashIsOnHover: false,
      onTrashPage: false,
      msgList: [],
      inputContent: "",
      timer: null,
    };
  },
  mounted() {
    (() => {
      let random = Math.floor(Math.random() * this.options.length);
      this.identity = this.options[random].value;
    })();
    this.getMsgs();
    this.timer = setInterval(() => {
      this.getMsgs();
    }, 1000 * 4);
  },
  methods: {
    momentAgo: function (e) {
      return moment(e).calendar();
    },
    clickOnPost: function () {
      this.postIsOnHover = true;
      this.inputBoxShown = !this.inputBoxShown;
      setTimeout(() => {
        this.postIsOnHover = false;
      }, 50);
    },
    // clickOnRefresh: function (e) {
    //   this.refreshIsOnHover = true
    //   setTimeout(() => {
    //     this.refreshIsOnHover = false
    //   }, 50);
    //   this.getMsgs()
    // },
    clickOnTrash: function () {
      this.onTrashPage = !this.onTrashPage;
      this.getMsgs();
    },
    clickOnSubmit: function () {
      this.submitIsOnHover = true;
      setTimeout(() => {
        this.submitIsOnHover = false;
      }, 50);
      this.onTrashPage = false;
      axios
        .post(`${HOSTNAME}/post-msg`, {
          content: `${this.identity}: ${this.inputContent}`,
        })
        .then(() => {
          this.getMsgs();
          this.inputContent = "";
        });
    },
    clickOnDelete: function (e) {
      axios
        .post(`${HOSTNAME}/remove-msg`, {
          msgId: e.target.id,
        })
        .then(() => {
          this.getMsgs();
        });
    },

    getForId: function (itemId) {
      return itemId;
    },

    getMsgs: function () {
      if (this.onTrashPage == true) {
        axios.get(`${HOSTNAME}/get-trash`).then((response) => {
          this.msgList = response.data.data;
        });
      } else {
        axios.get(`${HOSTNAME}/get-msgs`).then((response) => {
          this.msgList = response.data.data;
        });
      }
    },
  },
};
</script>

<style>
html {
  background-color: #f8f8f8;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

@media (prefers-color-scheme: dark) {
  html {
    background-color: #171717;
  }
  #app {
    background-color: #171717;
  }
}
</style>
