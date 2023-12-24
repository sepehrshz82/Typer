<script setup lang="ts">
import lessons from "~/assets/lessons";
import { ref } from "vue";
import { Switch, SwitchGroup, SwitchLabel } from '@headlessui/vue'
import { onClickOutside } from '@vueuse/core'
import ResultPopup from "@/components/ResultPopup.vue"
import HandHolder from "@/components/HandHolder.vue";
import Keyboard from "@/components/Keyboard.vue"
import type {TypeInfo} from "@/types/index"
import neutralLeft from "@/public/hands/NeutralLeft.svg"
import neutralRight from "@/public/hands/NeutralRight.svg"

// Define reactive variables
let timer: NodeJS.Timeout | undefined = undefined
const charIndex = ref<number>(0)
const isFirst = ref<boolean>(true)
const enabledDetail = ref<boolean>(true)
const isTyping = ref<boolean>(false)
const inputField = ref<string>("")
const inputFieldRef = ref()
const typedChar = ref<string>("")
const letterIndex = ref<number>(0)
const LessonBox = ref()
const route = useRouter()
const id = route.currentRoute.value.params?.id

const typeInfo = ref<TypeInfo>({
  resultWindow: false,
  startTime: 0,
  mistakes: 0,
  wpmTag: 0,
  cpmTag: 0
})

const focusInput = () => {
  inputFieldRef.value.focus()
}

onClickOutside(inputFieldRef, focusInput)

const loadLesson = ref<
  {
    letter: string
    active: boolean
    status: "correct" | "incorrect" | "nutrul"
  }[]
>(
  lessons[id - 1].value
  .split("")
  .map((v) => ({ letter: v, active: false, status: "nutrul" }))
  )

loadLesson.value[0].active = true

const currentLetters = computed(() => loadLesson.value.slice(letterIndex.value, letterIndex.value + 4))

const initTyping = () => {
  isLeftNeutral.value = false
  isRightNeutral.value = false
  typedChar.value = inputField.value.split("")[charIndex.value];
  console.log(inputField.value)
  console.log("key:" + typedChar)
  if (charIndex.value < loadLesson.value.length) {
    if (!isTyping.value) {
      timer = setInterval(initTimer, 1000);
      isTyping.value = true;
    }
    // if (typedChar.value == null || undefined) {
    //   if (charIndex.value > 0) {
    //     if (charIndex.value % 4 === 0) letterIndex.value -= 4
    //       charIndex.value--;
    //       if (loadLesson.value[charIndex.value].status === "incorrect") {
    //         typeInfo.value.mistakes--;
    //       }
    //       loadLesson.value[charIndex.value].status = "nutrul";
    //   }
    //   loadLesson.value[charIndex.value + 1].active = false;
    // }
    if (typedChar.value == null || undefined) {
      inputField.value += loadLesson.value[charIndex.value - 1]
    }
    else {
      if (loadLesson.value[charIndex.value].letter == typedChar.value) {
        if (isFirst.value) loadLesson.value[charIndex.value].status = "correct";
          charIndex.value++;
          loadLesson.value[charIndex.value - 1].active = false;
          isFirst.value = true
      }
      else {
        if(isFirst.value) {
          typeInfo.value.mistakes++;
          loadLesson.value[charIndex.value].status = "incorrect";
          isFirst.value = false
        }
        inputField.value = inputField.value.slice(0, -1)
      }
    }
    if (charIndex.value != loadLesson.value.length) {
      loadLesson.value[charIndex.value].active = true;
    }
    typeInfo.value.cpmTag = charIndex.value - typeInfo.value.mistakes;
  }
  if (charIndex.value == loadLesson.value.length) {
    loadLesson.value[charIndex.value - 1].active = false;
    typeInfo.value.resultWindow = true
    clearInterval(timer);
    inputField.value = "";
  }
  if (charIndex.value === letterIndex.value + 4) {
    letterIndex.value += 4
  }
}

const initTimer = () => {
  typeInfo.value.startTime++;
  typeInfo.value.startTime;
}

const resetGame = () => {
  typeInfo.value.resultWindow = false
  clearInterval(timer)
  typeInfo.value.startTime = 0
  charIndex.value = 0
  typeInfo.value.mistakes = 0
  isTyping.value = false
  inputField.value = ""
  typeInfo.value.cpmTag = 0
  letterIndex.value = 0
  loadLesson.value = lessons[id - 1].value.split("").map((v) => ({ letter: v, active: false, status: "nutrul" }))
  focusInput()
  loadLesson.value[0].active = true
}

const isLeftNeutral = ref(false)
const isRightNeutral = ref(true)

const getLeftHand = computed(()=>{
  return ("/hands/Key" + loadLesson.value[charIndex.value].letter.toUpperCase() + "Left.svg")
})

const getRightHand = computed(()=>{
  if (loadLesson.value[charIndex.value].letter === "/")
    return ("/hands/KeySlashRight.svg")
  else if (loadLesson.value[charIndex.value].letter === "[")
    return ("/hands/KeyOpenBracketRight.svg")
  else if (loadLesson.value[charIndex.value].letter === "]")
    return ("/hands/KeyCloseBracketRight.svg")
  else 
    return ("/hands/Key" + loadLesson.value[charIndex.value].letter.toUpperCase() + "Right.svg")
})

</script>

<template>
  <div class="flex justify-center">
  <div class="w-[780px] pt-6 p-[35px] bg-white">
    <!-- Lesson box -->
    <input autofocus @input="initTyping" ref="inputFieldRef" v-model="inputField" type="text" class="opacity-0 z-0 absolute" />
    <div class="rounded-xl border-2 border-gray-400 flex flex-col items-center">
      <!-- Lesson box header -->
      <div class="bg-gray-400 w-full h-14 rounded-t-lg flex justify-around items-center">
        <SwitchGroup as="div" class="flex items-center">
          <SwitchLabel as="span" class="mr-3">
            <span class="text-sm font-medium text-gray-900">Time and speed</span>
          </SwitchLabel>
          <Switch v-model="enabledDetail" :class="[enabledDetail ? 'bg-purple-600' : 'bg-gray-200', 'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none']">
            <span class="sr-only">Use setting</span>
            <span :class="[enabledDetail ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none relative inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out']">
              <span :class="[enabledDetail ? 'opacity-0 ease-out duration-100' : 'opacity-100 ease-in duration-200', 'absolute inset-0 flex h-full w-full items-center justify-center transition-opacity']" aria-hidden="true">
                <svg class="h-3 w-3 text-gray-400" fill="none" viewBox="0 0 12 12">
                  <path d="M4 8l2-2m0 0l2-2M6 6L4 4m2 2l2 2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </span>
              <span :class="[enabledDetail ? 'opacity-100 ease-in duration-200' : 'opacity-0 ease-out duration-100', 'absolute inset-0 flex h-full w-full items-center justify-center transition-opacity']" aria-hidden="true">
                <svg class="h-3 w-3 text-purple-600" fill="currentColor" viewBox="0 0 12 12">
                  <path d="M3.707 5.293a1 1 0 00-1.414 1.414l1.414-1.414zM5 8l-.707.707a1 1 0 001.414 0L5 8zm4.707-3.293a1 1 0 00-1.414-1.414l1.414 1.414zm-7.414 2l2 2 1.414-1.414-2-2-1.414 1.414zm3.414 2l4-4-1.414-1.414-4 4 1.414 1.414z" />
                </svg>
              </span>
            </span>
          </Switch>
        </SwitchGroup>
      </div>
      <!-- Progress bar -->
      <UProgress v-if="enabledDetail" class="w-11/12 h-16 mt-2" :value="Math.floor((charIndex / loadLesson.length) * 100)" color="purple" size="lg" indicator>
        <template #indicator="{ percent }">
          <div class="text-right" :style="{ width: `${percent}%` }">
            <span class="text-purple-700 text-lg transition-all ease-linear">
              {{ Math.floor(percent) }}%
            </span>
          </div>
        </template>
      </UProgress>
      <!-- Lesson text -->
      <div ref="LessonBox" :class="!enabledDetail ? 'mt-6' : ''" class="w-full flex items-center justify-around text-9xl mt-2 h-48 px-5">
        <span v-for="(word, index) in currentLetters" :key="index" class="flex justify-center w-1/5 py-7"
          :class="[
            word.active ? 'text-purple-600 border border-purple-600 bg-electric-violet-100 rounded-md' : 'bg-gray-100 rounded-md', 
            word.status === 'correct' ? 'text-green-600' : '', 
            word.status === 'incorrect' ? 'text-red-600 bg-pink-200' : ''
          ]">
          {{ word.letter }}
        </span>
      </div>
      <!-- Typing details -->
      <div class="mt-6 w-11/12 flex py-6 justify-center flex-row border-t-2 border-t-gray-400">
        <ul v-if="enabledDetail" class="flex items-center w-full">
          <li class="flex h-5 mr-5 list-none relative items-center">
            <p class="text-lg">Start Time:</p>
            <span class="block text-[20px] ml-2">{{ typeInfo.startTime }}s</span>
          </li>
          <li class="flex h-5 list-none relative items-center px-5 border-l-2 border-purple-600">
            <p class="text-lg">Mistakes:</p>
            <span class="block text-[20px] ml-2">{{ typeInfo.mistakes }}</span>
          </li>
          <li class="flex h-5 list-none relative items-center px-5 border-l-2 border-purple-600">
            <p class="text-lg">CPM:</p>
            <span class="block text-[20px] ml-2">{{ typeInfo.cpmTag }}</span>
          </li>
        </ul>
        <button @click="resetGame" class="outline-none border-none w-28 text-white py-2 text-sm font-semibold cursor-pointer rounded-md bg-purple-500 transition-all hover:bg-purple-600 active:scale-90">Try Again</button>
      </div>
    </div>
    <!-- Keyboard -->
    <div class="relative h-[360px] overflow-hidden">
      <div class="w-full z-10 absolute flex">
        <img @error="isLeftNeutral = true" class="h-96 flex justify-start scale-150 border-none" :src="isLeftNeutral ? neutralLeft : getLeftHand" />
        <img @error="isRightNeutral = true" class="h-96 scale-150 border-none object-cover" :src="isRightNeutral ? neutralRight : getRightHand" />
      </div>
      <Keyboard />
    </div>
    <!-- Result pop-up -->
    <ResultPopup :type-info="typeInfo" @try-again="resetGame" :isWpm="false" />
  </div>
  </div>
</template>

<style>
::selection {
  color: #fff;
  background: rgb(147, 51, 234);
}
</style>