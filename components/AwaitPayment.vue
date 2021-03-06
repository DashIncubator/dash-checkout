<template>
  <div>
    <v-card min-width="285px"
      ><v-card-title class="text-center mx-auto"
        ><span v-if="isPaid">Payment Received.</span>
        <span v-if="!isPaid">Waiting for Payment.</span></v-card-title
      >
      <!-- <v-spacer style="height: 40px;" /> -->
      <!-- <v-spacer v-if="!isPaid" style="height: 120px;" /> -->
      <v-card-text v-if="!isPaid" class="text-center" style="height: 120px;">
        <div style="height: 30px;"></div>
        <v-progress-circular indeterminate="" class="text-center" size="60" />
      </v-card-text>
      <svg
        v-if="isPaid"
        class="checkmark"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 52 52"
      >
        <circle class="checkmark__circle" cx="26" cy="26" r="25" fill="none" />
        <path
          class="checkmark__check"
          fill="none"
          d="M14.1 27.2l7.1 7.2 16.7-16.8"
        />
      </svg>
      <!-- <v-spacer style="height: 50px;" /> -->
      <v-card-text class="text-center">
        From: {{ $store.state.selectedItem.intentDoc.requesteeUserName }} <br />
        Amount: {{ $store.state.selectedItem.fiatAmount }}
        {{ $store.state.selectedItem.fiatSymbol }}<br />
        <span v-if="satoshisReceived > 0">
          Received {{ dashReceived }} / {{ dashRequested }} Dash</span
        >
        <span v-else>Waiting .. </span>
      </v-card-text>
    </v-card>
    <v-alert
      v-if="isPaid"
      outlined
      border="top"
      type="info"
      color="green"
      elevation="2"
      prominent=""
    >
      Your <b>{{ $store.state.selectedItem.name }}</b> voucher is:<br />
      <code>{{ uuid }}</code>
    </v-alert>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { mapActions } from 'vuex'
import { Unit } from '@dashevo/dashcore-lib'

// @ts-ignore
import { v4 as uuidv4 } from 'uuid'
const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms))

export default Vue.extend({
  data: () => {
    const data: {
      isPaid: boolean
      satoshisReceived: number
      satoshisRequested: number
    } = {
      isPaid: false,
      satoshisReceived: 0,
      satoshisRequested: -1,
    }
    return data
  },
  computed: {
    dashReceived() {
      return Unit.fromSatoshis(this.satoshisReceived).toBTC()
    },
    dashRequested() {
      return Unit.fromSatoshis(this.satoshisRequested).toBTC()
    },
    uuid() {
      return uuidv4()
    },
  },
  async mounted() {
    await this.pollWaitForPayment()
  },
  methods: {
    ...mapActions(['getAddressSummary']),
    async pollWaitForPayment(): Promise<void> {
      if (this.$store.state.selectedItem.requestDoc.encAddress === undefined) {
        // Need to reset local variables if user goes to step 1 and does another purchase
        this.isPaid = false
        this.satoshisReceived = 0
        this.satoshisRequested = -1

        await sleep(2000)
        this.pollWaitForPayment()
        return
      }

      const document = this.$store.state.selectedItem.requestDoc
      this.satoshisRequested = document.encSatoshis

      const summary = await this.getAddressSummary(document.encAddress)
      this.satoshisReceived = summary.totalBalanceSat

      if (this.satoshisReceived >= document.encSatoshis) {
        this.isPaid = true
        // setTimeout(() => {
        //   // this.waitingForPayment = false
        //   this.$router.push('/')
        // }, 5000)
      }
      await sleep(2000)
      this.pollWaitForPayment()
    },
  },
})
</script>

<style scoped>
*,
*:before,
*:after {
  box-sizing: border-box;
}

body {
  background: #012060;
}

.loadcontainer {
  position: absolute;
  top: 50%;
  /* top: 130px; */
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  -webkit-transform-style: preserve-3d;
  transform-style: preserve-3d;
  -webkit-perspective: 2000px;
  perspective: 2000px;
  -webkit-transform: rotateX(-30deg) rotateY(-45deg);
  transform: rotateX(-30deg) rotateY(-45deg);
}

.holder {
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  -webkit-transform-style: preserve-3d;
  transform-style: preserve-3d;
  -webkit-transform: translate3d(0em, 3em, 1.5em);
  transform: translate3d(0em, 3em, 1.5em);
}
.holder:last-child {
  -webkit-transform: rotateY(-90deg) rotateX(90deg) translate3d(0, 3em, 1.5em);
  transform: rotateY(-90deg) rotateX(90deg) translate3d(0, 3em, 1.5em);
}
.holder:first-child {
  -webkit-transform: rotateZ(-90deg) rotateX(-90deg) translate3d(0, 3em, 1.5em);
  transform: rotateZ(-90deg) rotateX(-90deg) translate3d(0, 3em, 1.5em);
}
.holder:nth-child(1) .box {
  background-color: #008de4;
}
.holder:nth-child(1) .box:before {
  background-color: #004e7e;
}
.holder:nth-child(1) .box:after {
  background-color: #006db1;
}
.holder:nth-child(2) .box {
  background-color: #787878;
}
.holder:nth-child(2) .box:before {
  background-color: #454545;
}
.holder:nth-child(2) .box:after {
  background-color: #5f5f5f;
}
.holder:nth-child(3) .box {
  background-color: #ffffff;
}
.holder:nth-child(3) .box:before {
  background-color: #cccccc;
}
.holder:nth-child(3) .box:after {
  background-color: #e6e6e6;
}

.box {
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  -webkit-transform-style: preserve-3d;
  transform-style: preserve-3d;
  -webkit-animation: ani-box 6s infinite;
  animation: ani-box 6s infinite;
  width: 3em;
  height: 3em;
}
.box:before,
.box:after {
  position: absolute;
  width: 100%;
  height: 100%;
  content: '';
}
.box:before {
  left: 100%;
  bottom: 0;
  -webkit-transform: rotateY(90deg);
  transform: rotateY(90deg);
  -webkit-transform-origin: 0 50%;
  transform-origin: 0 50%;
}
.box:after {
  left: 0;
  bottom: 100%;
  -webkit-transform: rotateX(90deg);
  transform: rotateX(90deg);
  -webkit-transform-origin: 0 100%;
  transform-origin: 0 100%;
}

@-webkit-keyframes ani-box {
  8.33% {
    -webkit-transform: translate3d(-50%, -50%, 0) scaleZ(2);
    transform: translate3d(-50%, -50%, 0) scaleZ(2);
  }
  16.7% {
    -webkit-transform: translate3d(-50%, -50%, -3em) scaleZ(1);
    transform: translate3d(-50%, -50%, -3em) scaleZ(1);
  }
  25% {
    -webkit-transform: translate3d(-50%, -100%, -3em) scaleY(2);
    transform: translate3d(-50%, -100%, -3em) scaleY(2);
  }
  33.3% {
    -webkit-transform: translate3d(-50%, -150%, -3em) scaleY(1);
    transform: translate3d(-50%, -150%, -3em) scaleY(1);
  }
  41.7% {
    -webkit-transform: translate3d(-100%, -150%, -3em) scaleX(2);
    transform: translate3d(-100%, -150%, -3em) scaleX(2);
  }
  50% {
    -webkit-transform: translate3d(-150%, -150%, -3em) scaleX(1);
    transform: translate3d(-150%, -150%, -3em) scaleX(1);
  }
  58.3% {
    -webkit-transform: translate3d(-150%, -150%, 0) scaleZ(2);
    transform: translate3d(-150%, -150%, 0) scaleZ(2);
  }
  66.7% {
    -webkit-transform: translate3d(-150%, -150%, 0) scaleZ(1);
    transform: translate3d(-150%, -150%, 0) scaleZ(1);
  }
  75% {
    -webkit-transform: translate3d(-150%, -100%, 0) scaleY(2);
    transform: translate3d(-150%, -100%, 0) scaleY(2);
  }
  83.3% {
    -webkit-transform: translate3d(-150%, -50%, 0) scaleY(1);
    transform: translate3d(-150%, -50%, 0) scaleY(1);
  }
  91.7% {
    -webkit-transform: translate3d(-100%, -50%, 0) scaleX(2);
    transform: translate3d(-100%, -50%, 0) scaleX(2);
  }
  100% {
    -webkit-transform: translate3d(-50%, -50%, 0) scaleX(1);
    transform: translate3d(-50%, -50%, 0) scaleX(1);
  }
}

@keyframes ani-box {
  8.33% {
    -webkit-transform: translate3d(-50%, -50%, 0) scaleZ(2);
    transform: translate3d(-50%, -50%, 0) scaleZ(2);
  }
  16.7% {
    -webkit-transform: translate3d(-50%, -50%, -3em) scaleZ(1);
    transform: translate3d(-50%, -50%, -3em) scaleZ(1);
  }
  25% {
    -webkit-transform: translate3d(-50%, -100%, -3em) scaleY(2);
    transform: translate3d(-50%, -100%, -3em) scaleY(2);
  }
  33.3% {
    -webkit-transform: translate3d(-50%, -150%, -3em) scaleY(1);
    transform: translate3d(-50%, -150%, -3em) scaleY(1);
  }
  41.7% {
    -webkit-transform: translate3d(-100%, -150%, -3em) scaleX(2);
    transform: translate3d(-100%, -150%, -3em) scaleX(2);
  }
  50% {
    -webkit-transform: translate3d(-150%, -150%, -3em) scaleX(1);
    transform: translate3d(-150%, -150%, -3em) scaleX(1);
  }
  58.3% {
    -webkit-transform: translate3d(-150%, -150%, 0) scaleZ(2);
    transform: translate3d(-150%, -150%, 0) scaleZ(2);
  }
  66.7% {
    -webkit-transform: translate3d(-150%, -150%, 0) scaleZ(1);
    transform: translate3d(-150%, -150%, 0) scaleZ(1);
  }
  75% {
    -webkit-transform: translate3d(-150%, -100%, 0) scaleY(2);
    transform: translate3d(-150%, -100%, 0) scaleY(2);
  }
  83.3% {
    -webkit-transform: translate3d(-150%, -50%, 0) scaleY(1);
    transform: translate3d(-150%, -50%, 0) scaleY(1);
  }
  91.7% {
    -webkit-transform: translate3d(-100%, -50%, 0) scaleX(2);
    transform: translate3d(-100%, -50%, 0) scaleX(2);
  }
  100% {
    -webkit-transform: translate3d(-50%, -50%, 0) scaleX(1);
    transform: translate3d(-50%, -50%, 0) scaleX(1);
  }
}

.checkmark__circle {
  stroke-dasharray: 166;
  stroke-dashoffset: 166;
  stroke-width: 2;
  stroke-miterlimit: 10;
  stroke: #7ac142;
  fill: none;
  animation: stroke 0.6s cubic-bezier(0.65, 0, 0.45, 1) forwards;
}

.checkspace {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: block;
  stroke-width: 2;
  margin: 10% auto;
}
.checkmark {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: block;
  stroke-width: 2;
  stroke: #fff;
  stroke-miterlimit: 10;
  margin: 10% auto;
  box-shadow: inset 0px 0px 0px #7ac142;
  animation: fill 0.4s ease-in-out 0.4s forwards,
    scale 0.3s ease-in-out 0.9s both;
}

.checkmark__check {
  transform-origin: 50% 50%;
  stroke-dasharray: 48;
  stroke-dashoffset: 48;
  animation: stroke 0.3s cubic-bezier(0.65, 0, 0.45, 1) 0.8s forwards;
}

@keyframes stroke {
  100% {
    stroke-dashoffset: 0;
  }
}
@keyframes scale {
  0%,
  100% {
    transform: none;
  }
  50% {
    transform: scale3d(1.1, 1.1, 1);
  }
}
@keyframes fill {
  100% {
    box-shadow: inset 0px 0px 0px 30px #7ac142;
  }
}
</style>
