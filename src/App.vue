<template>
  <transition name="modal">
    <div>
      <div class="modal-bg" v-if="showModal" @click="closeQuote"></div>
      <div class="quote-window" v-if="showModal" @click="modalQuoteAndReply">
        <div class="rc-infoCard">
          <div class="rc-infoDetail">
            {{modalQuote}}
          </div>
        </div>
      </div>
    </div>
  </transition>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <div class="container">
    <div style="text-align:left" class="rc-infoCard">
      <div class="rc-infoDetail">
        🤥 First Rule:
        <strong> <center> Be respectful. </center> </strong>
        <br>
        
        <!-- <strong> 📣 Ad: Gallinula is now in open beta! <br> </strong> <br>
        - 🗣 private message / group message <br>
        - 💬 starting a topic & vote. Reply & Repost <br> 
        - 📩 email notification <br>
        - 🤩 (finally!!) mobile app supported <br> <br>
        <a href="https://mp.weixin.qq.com/s/ju4JgQkTLeeXdoYhTkWz4Q"> Be sure to check this out! </a> -->
      </div>
    </div>
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
        <div v-if="replyQuote !== ''" @click="clearQuote">
          <div class="rc-infoCard">
            <div class="rc-infoDetail">
            <div style="text-align:left; font-size:13px">
                 You quoted: (Tap to cancel) <br> 
                {{replyQuote}}
            </div>
            </div>
          </div>
        </div>
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
      <div class="rc-infoCard" @click="quoteAndReply">
        <div class="delete">
          <div
            class="deleteTouch"
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
          <div class="msg">
            {{ item.msg }}
          </div>
          <div class="quoteMsg" v-if="item.quote_msg" @click="showQuote">
          {{ item.quote_msg}}
          </div>
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
    <div v-if="!onTrashPage" class="button" @click="toggleGetMore">🔽</div>
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

// const HOSTNAME = "http://10.200.68.152:3090";
const HOSTNAME = "https://thewallengine.gallinula.com";
// const HOSTNAME = "http://localhost:3090";

function getQueryString(name) { 
  let reg = `(^|&)${name}=([^&]*)(&|$)`
  let r = window.location.search.substr(1).match(reg); 
  if (r != null) return unescape(r[2]); return null; 
}



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
      allowOnHoverMsg: true,
      showModal: false,
      postIsOnHover: false,
      submitIsOnHover: false,
      inputBoxShown: false,
      // refreshIsOnHover: false,
      trashIsOnHover: false,
      onTrashPage: false,
      msgList: [],
      inputContent: "",
      timer: null,
      modalQuote: "",
      replyQuote: "",
      afterTime: 0,
    };
  },
  created() {
    
  },
  mounted() {
    let token = getQueryString('token');
    if (token) {
        (() => {
        let random = Math.floor(Math.random() * this.options.length);
        this.identity = this.options[random].value;
      })();
      this.getMsgs();
      this.timer = setInterval(() => {
        this.getMsgs();
      }, 1000 * 4);  
    }
    else {
      // window.location.href = `${HOSTNAME}/welcome.html`;
      window.location.href = `https://thewall.gallinula.com/welcome.html`;
    }
    // let token = getQueryString('token');
    // if (!token) {
    //   window.location.href = `${HOSTNAME}/verify`;
    // }
    // console.log(token);
    // this.afterTime = new Date().toISOString();
    
  },
  methods: {
    modalQuoteAndReply(e) {
      if (this.onTrashPage || e.target.classList.contains("quoteMsg") || e.target.classList.contains("deleteTouch")) {
        return
      }
      this.closeQuote();
      this.replyQuote = this.modalQuote;
      this.inputBoxShown = true;
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: "smooth",
      });
    },
    toggleGetMore() {
      this.getSomeMoreMsgs();
    },
    clearQuote () {
      this.replyQuote = '';
    },
    quoteAndReply(e) {
      // console.log(e.currentTarget)
      if (this.onTrashPage || e.target.classList.contains("quoteMsg") || e.target.classList.contains("deleteTouch")) {
        return
      }
      this.inputBoxShown = true;
      this.replyQuote = e.currentTarget.querySelector(".rc-infoDetail .msg").innerText;
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: "smooth",
      });
      // console.log(e.currentTarget.classList);
      // let bgColor = e.currentTarget.style;
      // console.log(bgColor);
      // e.currentTarget.style.backgroundColor = "rgb(125,30,30)";
    },
    showQuote: function(e) {
      // console.log(e.target);
      this.showModal = true;
      this.modalQuote = e.target.innerText;
    },
    closeQuote: function() {
      this.showModal = false;
    },
    momentAgo: function (e) {
      return moment(e).calendar();
    },
    clickOnPost: function () {
      window._paq.push(['setCustomDimension', 1, 'clickOnPost'])
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
      window._paq.push(['setCustomDimension', 1, 'clickOnTrash']);
      this.onTrashPage = !this.onTrashPage;
      this.getMsgs();
    },
    clickOnSubmit: function () {
      window._paq.push(['setCustomDimension', 1, 'post-msg']);
      this.submitIsOnHover = true;
      setTimeout(() => {
        this.submitIsOnHover = false;
      }, 50);
      this.onTrashPage = false;
      axios
        .post(`${HOSTNAME}/post-msg`, {
          content: `${this.identity}: ${this.inputContent}`,
          quote_msg: `${this.replyQuote}`,
          token: `${getQueryString('token')}`,
        })
        .then(() => {
          this.getMsgs();
          this.inputContent = "";
          this.replyQuote = "";
        });
    },
    clickOnDelete: function (e) {
      window._paq.push(['setCustomDimension', 1, 'remove-msg']);
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
    getSomeMoreMsgs: function () {
      if (!this.onTrashPage) {
        axios.get(`${HOSTNAME}/get-some-more-msgs?beforeTime=${this.afterTime}&num=10`).then((response) => {
          // console.log(response.data);
          this.afterTime = response.data.data;
          this.getMsgs();
        })
      }
    },
    getMsgs: function () {
      if (this.onTrashPage == true) {
        axios.get(`${HOSTNAME}/get-trash`).then((response) => {
          this.msgList = response.data.data;
        });
      } else {
        axios.get(`${HOSTNAME}/get-msgs?afterTime=${this.afterTime}`, ).then((response) => {
          // console.log("payload", response.data.data);
          this.msgList = response.data.data;
          this.afterTime = response.data.data.at(-1).time;
          // console.log(this.afterTime);
        });
      }
    },
  },
};
</script>

<style>
html {
  background-color: #f8f8f8;
  -webkit-filter: grayscale(100%);
  filter: grayscale(100%);
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
