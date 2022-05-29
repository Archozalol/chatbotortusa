<script setup>
import { ref } from "vue";
import messages from "./components/messages.js";
import Message from "./components/Message.vue";
import TypingIndicator from "./components/TypingIndicator.vue";
fetch("http://localhost:8000/restart");
const message = ref("");
const loadingMessage = ref(false);
const currentCourse = ref("");
const addingQuestion = ref(false);

const question = ref("")
const answer = ref("")
const password = ref("")

async function getCurrentCourse() {
  let ans = await fetch("http://localhost:8000/course");
  currentCourse.value = await ans.json();
  return currentCourse.value;
}
async function addQuestion() {
  addingQuestion.value = false;
  messages.value.push({ message: "Pievieno jautājumu, lūdzu uzgaidiet", fromUser: false });
  loadingMessage.value = true;
  await fetch("http://localhost:8000/add", {
    method: "POST",
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      question: question.value,
      answer: answer.value,
      password: password.value
    })
  });
  loadingMessage.value = false;
  await train(200);
  messages.value.push({ message: ansText, fromUser: false });
}

async function train(count = 200) {
  messages.value.push({ message: "Modeļa apmācība uzsākta, lūdzu uzgaidiet", fromUser: false });
  loadingMessage.value = true;
  let ans = await fetch("http://localhost:8000/train?c=" + count);
  let ansText = await ans.json();
  loadingMessage.value = false;
  messages.value.push({ message: ansText, fromUser: false });
}

async function sendMessage() {
  loadingMessage.value = true;
  const messageText = message.value.toLowerCase();
  messages.value.push({ message: messageText, fromUser: true });
  message.value = "";
  setTimeout(async () => {
    if (messageText) {
      let ans = await fetch("http://localhost:8000/ask?q=" + messageText);
      let ansText = await ans.json();
      messages.value.push({ message: ansText, fromUser: false });
      getCurrentCourse();
      loadingMessage.value = false;
      // message.value = "";
    }
    setTimeout(() => {
      const container = document.getElementById("messageBox");
      container.scrollTop = container.scrollHeight;
    }, 100);
  }, 1000);

}
function checkIfEnter(event) {
  if (event.keyCode === 13) {
    event.preventDefault();
    sendMessage();
  }
}
</script>

<template>
  <form v-if="addingQuestion" class="addQuestion" @submit="addQuestion">
    <input type="text" v-model="question" placeholder="Enter your question">
    <textarea type="text" v-model="answer" placeholder="Enter your answer" rows="3"></textarea>
    <input type="password" v-model="password" placeholder="Enter Password">
    <button> Add question </button>
  </form>
  <header>
    <h1>ORTUSA</h1>
    <div id="menuToggle">
      <!--
    A fake / hidden checkbox is used as click reciever,
    so you can use the :checked selector on it.
    -->
      <input type="checkbox" />

      <!--
    Some spans to act as a hamburger.
    
    They are acting like a real hamburger,
    not that McDonalds stuff.
    -->
      <span></span>
      <span></span>
      <span></span>

      <ul id="menu">
        <li>{{ currentCourse || "No course set" }}</li>
        <li @click="addingQuestion = !addingQuestion">Add question</li>
        <li @click="train(250)">Train</li>
      </ul>
    </div>
    <hr />
  </header>

  <main>
    <div id="messageBox">
      <Message v-for="message in messages" v-bind:message="message.message" v-bind:fromUser="message.fromUser" />
      <TypingIndicator v-if="loadingMessage" />
    </div>
    <div class="inputBox">
      <textarea type="text" v-model="message" v-on:keypress="checkIfEnter"></textarea>
      <button v-on:click="sendMessage()">Send</button>
    </div>
  </main>
</template>

<style>
:root {
  --vt-c-white: #ffffff;
  --vt-c-white-soft: #f8f8f8;
  --vt-c-white-mute: #f2f2f2;

  --vt-c-black: #181818;
  --vt-c-black-soft: #222222;
  --vt-c-black-mute: #282828;

  --vt-c-indigo: #2c3e50;

  --vt-c-divider-light-1: rgba(60, 60, 60, 0.29);
  --vt-c-divider-light-2: rgba(60, 60, 60, 0.12);
  --vt-c-divider-dark-1: rgba(84, 84, 84, 0.65);
  --vt-c-divider-dark-2: rgba(84, 84, 84, 0.48);

  --vt-c-text-light-1: var(--vt-c-indigo);
  --vt-c-text-light-2: rgba(60, 60, 60, 0.66);
  --vt-c-text-dark-1: var(--vt-c-white);
  --vt-c-text-dark-2: rgba(235, 235, 235, 0.64);
}

/* semantic color variables for this project */
:root {
  --color-background: var(--vt-c-white);
  --color-background-soft: var(--vt-c-white-soft);
  --color-background-mute: var(--vt-c-white-mute);

  --color-border: var(--vt-c-divider-light-2);
  --color-border-hover: var(--vt-c-divider-light-1);

  --color-heading: var(--vt-c-text-light-1);
  --color-text: var(--vt-c-text-light-1);

  --section-gap: 160px;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-background: var(--vt-c-black);
    --color-background-soft: var(--vt-c-black-soft);
    --color-background-mute: var(--vt-c-black-mute);

    --color-border: var(--vt-c-divider-dark-2);
    --color-border-hover: var(--vt-c-divider-dark-1);

    --color-heading: var(--vt-c-text-dark-1);
    --color-text: var(--vt-c-text-dark-2);
  }
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  position: relative;
  font-weight: normal;
}

body {
  color: var(--color-text);
  background: var(--color-background);
  transition: color 0.5s, background-color 0.5s;
  line-height: 1.6;
  font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Oxygen, Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
  font-size: 15px;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.addQuestion {
  position: fixed;
  top: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  gap: 25px;
  background-color: #fff;
  z-index: 1;
}

.addQuestion * {
  width: 80%;
  min-height: 42px;
  border: 2px solid #232323;
  border-radius: 8px;
  font-size: 18px;
  resize: none;
}

#messageBox {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding-top: 10px;
  padding-bottom: 10px;
  overflow-y: scroll;
}

body,
html,
#app {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
}

.inputBox {
  display: flex;
}

.inputBox button {
  flex-grow: 2;
  margin-left: 10px;
  margin-right: 10px;
}

.inputBox textarea {
  flex-grow: 3;
  margin-left: 10px;
  box-sizing: border-box;
  padding-top: 5px;
  padding-left: 5px;
}

main {
  height: 100%;
  max-height: 90%;
  padding-bottom: 10px;
}

header {
  margin: 0 auto;
  max-height: 10%;
}

h1 {
  text-align: center;
}


/*
 * Made by Erik Terwan
 * 24th of November 2015
 * MIT License
 *
 *
 * If you are thinking of using this in
 * production code, beware of the browser
 * prefixes.
 */

#menuToggle {
  display: block;
  position: absolute;
  top: 16px;
  right: 16px;

  z-index: 1;

  -webkit-user-select: none;
  user-select: none;
}

#menuToggle a {
  text-decoration: none;
  color: #232323;

  transition: color 0.3s ease;
}

#menuToggle a:hover {
  color: tomato;
}


#menuToggle input {
  display: block;
  width: 40px;
  height: 32px;
  position: absolute;
  top: -7px;
  left: -5px;

  cursor: pointer;

  opacity: 0;
  /* hide this */
  z-index: 2;
  /* and place it over the hamburger */

  -webkit-touch-callout: none;
}

/*
 * Just a quick hamburger
 */
#menuToggle span {
  display: block;
  width: 33px;
  height: 4px;
  margin-bottom: 5px;
  position: relative;

  background: #cdcdcd;
  border-radius: 3px;

  z-index: 1;

  transform-origin: 4px 0px;

  transition: transform 0.5s cubic-bezier(0.77, 0.2, 0.05, 1.0),
    background 0.5s cubic-bezier(0.77, 0.2, 0.05, 1.0),
    opacity 0.55s ease;
}

#menuToggle span:first-child {
  transform-origin: 0% 0%;
}

#menuToggle span:nth-last-child(2) {
  transform-origin: 0% 100%;
}

/* 
 * Transform all the slices of hamburger
 * into a crossmark.
 */
#menuToggle input:checked~span {
  opacity: 1;
  transform: rotate(45deg) translate(-2px, -1px);
  background: #232323;
}

/*
 * But let's hide the middle one.
 */
#menuToggle input:checked~span:nth-last-child(3) {
  opacity: 0;
  transform: rotate(0deg) scale(0.2, 0.2);
}

/*
 * Ohyeah and the last one should go the other direction
 */
#menuToggle input:checked~span:nth-last-child(2) {
  transform: rotate(-45deg) translate(0, -1px);
}

/*
 * Make this absolute positioned
 * at the top left of the screen
 */
#menu {
  position: absolute;
  width: 300px;
  right: -16px;
  margin: -100px 0 0 -50px;
  padding: 50px;
  padding-top: 125px;

  background: #ededed;
  list-style-type: none;
  -webkit-font-smoothing: antialiased;
  /* to stop flickering of text in safari */

  transform-origin: 0% 0%;
  transform: translate(100%, 0);

  transition: transform 0.5s cubic-bezier(0.77, 0.2, 0.05, 1.0);
}

#menu li {
  padding: 10px 0;
  font-size: 22px;
}

/*
 * And let's slide it in from the left
 */
#menuToggle input:checked~ul {
  transform: none;
}
</style>
