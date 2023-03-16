<script setup>
import { storeToRefs } from 'pinia';

import ChatIcon from '~/components/Icons/ChatIcon.vue';
import BarsIcon from '~/components/Icons/BarsIcon.vue';

import { usePresetsStore } from '~/stores/presets';

defineProps({
  presetName: {
    type: String,
    required: true,
  },
  setClientToUse: {
    type: Function,
    required: true,
  },
  setIsClientSettingsModalOpen: {
    type: Function,
    required: true,
  },
});
</script>

<template>
  <div
    class="flex flex-col absolute bottom-full w-full overflow-hidden mb-2 rounded-lg"
  >
    <div
      class="flex items-center justify-start gap-2 shadow-sm bg-white/[15%] text-white/80 text-sm backdrop-blur rounded-t py-1 px-3"
    >
      Presets
    </div>
    <div
      class="flex flex-col items-stretch bg-white/10 backdrop-blur-sm overflow-auto max-h-[160px]"
    >
      <div class="w-full flex flex-row">
        <button
          class="px-3 py-1 flex-1 flex flex-row items-center transition ease-in-out hover:bg-white/20 text-sm"
          :class="{ 'font-bold': presetName === 'chat' }"
          @click="setClientToUse('chat')"
        >
          <ChatIcon
            class="h-9 py-2 pr-2 rounded-lg"
            :class="{ 'text-blue-500': presetName === 'chat' }"
          />
          Chat with conversation
        </button>
        <button
          class="hover:bg-white/20 px-3 py-1 flex items-center transition ease-in-out"
          @click="setIsClientSettingsModalOpen(true, 'chat')"
        >
          <Icon class="w-5 h-5 text-white/70" name="bx:bxs-cog" />
        </button>
      </div>
      <div class="w-full flex flex-row">
        <button
          class="w-full px-3 py-1 flex flex-row items-center transition ease-in-out hover:bg-white/20 border-t border-white/5 text-sm"
          :class="{ 'font-bold': presetName === 'compose' }"
          @click="setClientToUse('compose')"
        >
          <BarsIcon
            class="h-9 py-2 pr-2 rounded-lg"
            :class="{ 'text-blue-500': presetName === 'compose' }"
          />
          Compose a blog/article
        </button>
        <button
          class="hover:bg-white/20 px-3 py-1 flex items-center transition ease-in-out border-t border-white/5"
          @click="setIsClientSettingsModalOpen(true, 'compose')"
        >
          <Icon class="w-5 h-5 text-white/70" name="bx:bxs-cog" />
        </button>
      </div>
    </div>
  </div>
</template>
