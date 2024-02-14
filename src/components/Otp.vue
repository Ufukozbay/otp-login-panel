<script>
import { defineComponent, ref, computed, watch } from "vue";
import axios from "axios";
export default defineComponent({
  name: "LoginPanel",
  setup() {
    //LoginData variables
    const loginData = ref({
      username: "",
      password: "",
    });
    const otpData = ref({
      username: "",
      otpCode: "",
    });
    const bannerText = ref("");
    const otpInput = ref(Array.from({ length: 6 }, () => ""));
    const otpRemainingTime = ref(120);
    const inputRef = ref(null);
    const currentStatus = ref("loginForm");
    const disableInput = ref(false);

    const sendLoginForm = function(){
        if (loginData.value.username == "ufuk" && loginData.value.password == "123123") {
            console.log("login") 
            currentStatus.value = "otpForm" 
            console.log(currentStatus.value) 
            bannerText.value = "";
            disableInput.value = false; 
            otpInput.value = Array.from({ length: 6 }, () => "");
            remainingTimer();     
        }
        else{
            if (loginData.value.username != "ufuk") {
                bannerText.value = "userNotFound"                
            }
            else if (loginData.value.password != "123123") {
                bannerText.value = "wrongPassword"                
            }      

        }
    }
    const intervalId = ref(null);
    const remainingTimer = () => {
      otpRemainingTime.value = 120;
      intervalId.value = setInterval(() => {
        if (otpRemainingTime.value > 0) {
          otpRemainingTime.value--;
        } else {
          bannerText.value = "Expired";
          disableInput.value = true;
          stopRemainingTimer();
        }
      }, 1000);
    };
    const stopRemainingTimer = () => {
      clearInterval(intervalId.value);
    };
    watch(currentStatus, (a, oldStatus) => {
      if (oldStatus == "otpForm") {
        stopRemainingTimer();
      }
    });
    const progressBarValue = computed(() => {
      return (otpRemainingTime.value / 120) * 100
    });
    //Focus next input handle function
    const handleInput = (index, event) => {
      if (event.data == "" || event.data == null) {
        return;
      }
      let input = otpInput.value[index];
      if (typeof input == "number") {
        input %= 10;
      }
      if (typeof input == "string") {
        input = input.trim();
        if (input.length > 1) {
          input = input.slice(-1);
        }
      }
      otpInput.value[index] = input;
      navigateNextInput(event);
      if (otpInput.value.find((x) => x == "") == null) {
        sendForm();
      }
    };
    //Paste input handle function
    function handlePaste(e) {
      e.preventDefault();
      if (e.clipboardData.getData("text").length === 6) {
        for (let i = 0; i < e.clipboardData.getData("text").length; i++) {
          otpInput.value[i] = e.clipboardData
            .getData("text")
            .toString()
            .split("")[i];
        }
        sendForm();
      }
    }
    const handleBackspace = (index, event) => {
      if (event.key === "Backspace" && otpInput.value[index] === "") {
        navigateNextInput(event, false);
      }
    };
    function navigateNextInput(event, moveNext = true) {
      if (event?.target != null) {
        const itemList = [
          ...event.target.parentElement.querySelectorAll("input"),
        ];
        const currentIndex = itemList.indexOf(event.target);
        if (currentIndex > -1) {
          if (moveNext && itemList.length - 1 > currentIndex) {
            itemList[currentIndex + 1].focus();
          } else if (!moveNext && currentIndex > 0) {
            itemList[currentIndex - 1].focus();
          }
        }
      }
    }

    const sendForm = function(){
        console.log("sendForm") 
    }

    return {
      loginData,
      otpInput,
      otpData,
      bannerText,
      otpRemainingTime,
      inputRef,
      currentStatus,
      intervalId,
      sendLoginForm,
      isPwd: ref(true),
      progressBarValue,
      handleInput,
      handlePaste,
      handleBackspace,
      sendForm,
      disableInput,
    };
  },
});
</script>
<template>
  <div class="card">
    <div class="card-body">
      <div v-if="currentStatus === 'loginForm'" class="login-form">
        <form @submit.prevent="sendLoginForm">
          <div class="mb-3">
            <label class="form-label">Username</label>
            <input
              type="text"
              class="form-control"
              v-model="loginData.username"
              lazy-rules
              :rules="[(val) => !!val || $t('login.usernameValidation')]"
            />
          </div>
          <div class="mb-3">
            <label class="form-label">Password</label>
            <input
              type="password"
              class="form-control"
              v-model="loginData.password"
              lazy-rules
              :rules="[(val) => !!val || $t('login.passwordValidation')]"
              :type="isPwd ? 'password' : 'text'"
            />
          </div>
          <button @click="remainingTimer" type="submit" class="btn btn-primary mb-3">
            Submit
          </button>
          <div
            v-if="bannerText == 'userNotFound'"
            class="alert alert-danger"
            role="alert"
          >
          userNotFound
          </div>
          <div
            v-if="bannerText == 'wrongPassword'"
            class="alert alert-danger"
            role="alert"
          >
          wrongPassword
          </div>
        </form>
      </div>
      <div v-if="currentStatus === 'otpForm'" class="otp-form">
        <div class="mb-3">
          <form @submit.prevent="sendForm">
            <label class="form-label">Username</label>
            <div class="d-flex my-2">
              <input
                v-for="(input, index) in otpInput"
                v-model="otpInput[index]"
                @input="handleInput(index, $event)"
                @paste="handlePaste($event)"
                @keydown="handleBackspace(index, $event)"
                ref="inputRef"
                :key="index"
                :disabled="disableInput"
                type="text"
                class="form-control"
              />
            </div>

            <div class="progress">
                <div class="d-flex justify-content-center w-100"><p>Remaining Time : {{ otpRemainingTime }}</p></div>
              <div
                class="progress-bar"
                role="progressbar"
                aria-valuenow="0"
                aria-valuemin="0"
                :aria-valuemax="progressBarValue"
              ></div>
            </div>
            <div
              v-if="bannerText == 'wrongOtpCode'"
              class="alert alert-danger"
              role="alert"
            >
              This is a danger alert—check it out! 
            </div>
            <button class="btn btn-primary my-2" @click="currentStatus = 'loginForm'">Back</button>
            <div
              v-if="bannerText == 'wrongOtpCode'"
              class="alert alert-danger"
              role="alert"
            >
              This is a danger alert—check it out!
            </div>
            <div
              v-if="bannerText == 'Expired'"
              class="alert alert-danger"
              role="alert"
            >
              This is a danger alert—check it out! {{ otpRemainingTime }}
              <button class="btn btn-primary" @click="sendLoginForm()">Send Again</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="scss" scoped>
.card {
  width: 350px;
  .card-body {
    .otp-form {
      input {
        height: 50px;
        text-align: center;
        margin: 2px;
        font-size: 22px;
      }
    }
  }
}
</style>
