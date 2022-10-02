<script setup lang="ts">
import * as tf from '@tensorflow/tfjs'
const model = await tf.loadLayersModel('./tfjs_for_vuejs/model.json')
const user = useUserStore()
const name = $ref(user.savedName)

const router = useRouter()
const success = ref(false)
const mortgage = ref()
const income = ref()
const ratespread = ref()
const assets = ref()
const race = ref()
const date = ref()
const onwerOccupied = ref()
const race_calc = ref()
const usPerson = ref()
const message = ref()
const age = ref()

function sleep(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms)
  })
}

const predict = () => {
  if (race.value === 'Yes')
    race_calc.value = 1
  else
    race_calc.value = 0

  message.value = null
  // knowledge engineering
  if (usPerson.value == 'Yes') {
    message.value = "Sorry, we don't accept US Persons."
    return
  }
  
  if (onwerOccupied.value == 'Yes') {
    if (mortgage.value *0.1 > assets.value) {
      message.value = "Sorry, you need at least 10% of your mortgage in liquid assest."
      return
    }
      
    if (age.value > 60) {
      if (mortgage.value*(2/3) > (income.value*0.3*15)) {
        message.value = `Sorry, you need to be able to amortize your mortgage with 30% of your salary over the course of 15 years.`
        return
      }
    } else {
      if (mortgage.value*(2/3) > (income.value*15)) {
        message.value = `Sorry, you need to be able to amortize 2/3 of your mortgage with your salary over the course of 15 years.`
        return
      }
    }
  } else {
    if (mortgage.value *0.25 > assets.value) {
      message.value = "Sorry, you need at least 25% of your mortgage in liquid assest."
      return
    }
    if (age.value > 60) {
      if (mortgage.value*(2/3) > (income.value*0.3*10)) {
        message.value = `Sorry, you need to be able to amortize your mortgage with 30% of your salary over the course of 10 years.`
        return
      }
    } else {
      if (mortgage.value*(2/3) > (income.value*10)) {
        message.value = `Sorry, you need to be able to amortize 2/3 of your mortgage with your salary over the course of 10 years.`
        return
      }
    }
  }
  

  const input = tf.tensor([mortgage.value * 0.0000100002000040000804686633403828288635395438177511096000671386718750, income.value * 0.0001000100010001000132688411814463336213520960882306098937988281250000, ratespread * 0.0732064421669106901724433100753230974078178405761718750000000000000000, 1.0, date.value * 0.5000000000000000000000000000000000000000000000000000000000000000000000])
  const input2 = input.reshape([1, 5])
  const output = model.predict(input2).dataSync()
  console.log(output)
  if (output[0] > 0.5) {
    success.value = true
    sleep(2000).then(() => {
      success.value = false
      message.value = "Congratulations, you are approved!"
    })
  }
  else {
    success.value = false
    message.value = "Sorry, you are not approved."
  }
}
</script>

<template>
  <div v-if="success" class="svg-container flex justify-center items-center  ">
    <svg class="ft-green-tick" xmlns="http://www.w3.org/2000/svg" height="100" width="100" viewBox="0 0 48 48" aria-hidden="true">
      <circle class="circle" fill="#5bb543" cx="24" cy="24" r="22" />
      <path class="tick" fill="none" stroke="#FFF" stroke-width="6" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" d="M14 27l5.917 4.917L34 17" />
    </svg>
  </div>
  <div class="flex justify-center items-center h-2xl">
    <div class="w-full max-w-lg grid ml-12">
      <div class="flex flex-wrap -mx-3 mb-6">
        <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-first-name">
            Mortgage value
          </label>
          <input id="grid-first-name" v-model="mortgage" class=" appearance-none block w-full bg-gray-200 text-gray-700 border  rounded py-3 px-4 mb-3 leading-tight focus:outline-none focus:bg-white" type="number" placeholder="Value">
        </div>
        <div class="w-full md:w-1/2 px-3">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-last-name">
            Your income
          </label>
          <input id="grid-last-name" v-model="income" class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white focus:border-gray-500" type="number" placeholder="Value">
        </div>
      </div>
      <div class="flex flex-wrap -mx-3 mb-6">
        <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-rate-spread">
            Rate Spread
          </label>
          <input id="grid-rate-spread" v-model="ratespread" class=" appearance-none block w-full bg-gray-200 text-gray-700 border  rounded py-3 px-4 mb-3 leading-tight focus:outline-none focus:bg-white" type="number" placeholder="Value">
        </div>
        <div class="w-full md:w-1/2 px-3">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-assets">
            Your Assets
          </label>
          <input id="grid-assets" v-model="assets" class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white focus:border-gray-500" type="number" placeholder="Value">
        </div>
      </div>
      <div class="flex flex-wrap -mx-3 mb-2">
        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-state">
            White Person          </label>
          <div class="relative">
            <select id="grid-state" v-model="race" class="block appearance-none w-full bg-gray-200 border border-gray-200 text-gray-700 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500">
              <option>No</option>
              <option>Yes</option>
            </select>
            <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
              <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z" /></svg>
            </div>
          </div>
        </div>
        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-zip">
            application_date_indicator
          </label>
          <input id="grid-zip" v-model="date" class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white focus:border-gray-500" type="text" placeholder="1">
        </div>
      </div>
      <div class="flex flex-wrap -mx-3 mb-2">
        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-onwerOccupied">
            Owner-occupied
          </label>
          <div class="relative">
            <select id="grid-onwerOccupied" v-model="onwerOccupied" class="block appearance-none w-full bg-gray-200 border border-gray-200 text-gray-700 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500">
              <option>No</option>
              <option>Yes</option>
            </select>
            <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
              <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z" /></svg>
            </div>
          </div>
        </div>

        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-us-person">
            US-Person
          </label>
          <div class="relative">
            <select id="grid-us-person" v-model="usPerson" class="block appearance-none w-full bg-gray-200 border border-gray-200 text-gray-700 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500">
              <option>No</option>
              <option>Yes</option>
            </select>
            <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
              <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z" /></svg>
            </div>
          </div>
        </div>
        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
          <label class="block dark:text-white uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="grid-age">
            Age
          </label>
          <input id="grid-age" v-model="age" class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white focus:border-gray-500" type="text" placeholder="1">
        </div>
      </div>
      <button class="btn btn-primary" @click="predict" @submit.prevent="predict">
        click me
      </button>
      <p>{{ message }}</p>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>

<style scoped>
@supports (animation: grow .5s cubic-bezier(.25, .25, .25, 1) forwards) {
     .tick {
        stroke-opacity: 0;
        stroke-dasharray: 29px;
        stroke-dashoffset: 29px;
        animation: draw .5s cubic-bezier(.25, .25, .25, 1) forwards;
        animation-delay: .6s
    }

    .circle {
        fill-opacity: 0;
        stroke: #219a00;
        stroke-width: 16px;
        transform-origin: center;
        transform: scale(0);
        animation: grow 1s cubic-bezier(.25, .25, .25, 1.25) forwards;
    }
}

@keyframes grow {
    60% {
        transform: scale(.8);
        stroke-width: 4px;
        fill-opacity: 0;
    }
    100% {
        transform: scale(.9);
        stroke-width: 8px;
        fill-opacity: 1;
        fill: #219a00;
    }
}

@keyframes draw {
    0%, 100% { stroke-opacity: 1; }
    100% { stroke-dashoffset: 0; }
}

:root {
  --theme-color: var(--color-purple);
}

body {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}
</style>
