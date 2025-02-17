<script setup>
import { storeToRefs } from 'pinia';
import { useAppStore } from '~/stores/app';
import BingIcon from '~/components/Icons/BingIcon.vue';
import GPTIcon from '~/components/Icons/GPTIcon.vue';
import ChatIcon from '~/components/Icons/ChatIcon.vue';
import BarsIcon from '~/components/Icons/BarsIcon.vue';

const appStore = useAppStore();
const {
    isMobileMenu,
    isMenuOpen,
    isMenuOpening,
    isMenuClosing,
} = storeToRefs(appStore);

const conversationsStore = useConversationsStore();
const {
    currentConversationId,
    conversations,
    processingController,
} = storeToRefs(conversationsStore);
const {
    startNewConversation,
    setCurrentConversationId,
    deleteConversation,
    clearConversations,
} = conversationsStore;

// sort conversations by updatedAt timestamp
const sortedConversations = computed(() => Object.values(conversations.value).sort((a, b) => b.updatedAt - a.updatedAt));

const isConfirmingClear = ref(false);

const startNewConversationHandler = () => {
    startNewConversation();
    if (isMobileMenu.value) {
        isMenuOpen.value = false;
    }
};

const setCurrentConversationIdHandler = (conversationId) => {
    setCurrentConversationId(conversationId);
    if (isMobileMenu.value) {
        isMenuOpen.value = false;
    }
};

const deleteConversationHandler = (conversationId) => {
    deleteConversation(conversationId);
};

const clearConversationsHandler = () => {
    if (!isConfirmingClear.value) {
        isConfirmingClear.value = setTimeout(() => {
            isConfirmingClear.value = false;
        }, 3000);
        return;
    }
    clearConversations();
    clearTimeout(isConfirmingClear.value);
    isConfirmingClear.value = false;
};
</script>

<template>
    <div
        class="
            flex flex-col bg-purple-300/10 flex-1 max-w-xs shadow-lg
            absolute top-0 left-0 h-screen z-50 w-full
            backdrop-blur-lg
            transition-all duration-300 ease-in-out
        "
        :class="{
            'translate-x-0 absolute lg:static': isMenuOpen,
            '!absolute': isMenuOpening,
            '-translate-x-full': !isMenuOpen,
        }"
    >
        <!-- close button -->
        <Transition name="fade">
            <div
                v-if="isMenuOpen || isMenuClosing"
                class="absolute top-0 -right-8 bottom-0 flex items-center lg:hidden"
            >
                <button
                    @click="isMenuOpen = false"
                    class="
                    bg-purple-300/10 rounded-r backdrop-blur-lg
                    text-white/50 hover:text-white/80
                    focus:outline-none z-10
                    transition duration-300 ease-in-out
                "
                >
                    <Icon name="bx:bx-chevron-left" class="w-8 h-8"/>
                </button>
            </div>
        </Transition>
        <!-- Chat threads -->
        <div class="p-3 pb-0">
            <!-- New Chat -->
            <button
                @click="startNewConversationHandler"
                class="
                    flex flex-row items-center w-full bg-white/5 rounded shadow p-3 mb-3
                    transition duration-300 ease-in-out
                    hover:bg-white/10
                "
                :disabled="!!processingController"
                :class="{ 'cursor-not-allowed': !!processingController }"
            >
                    <span class="flex flex-col flex-1 text-left">
                        <span class="text-sm font-bold text-white/80">New Chat</span>
                        <span class="text-xs text-white/60">Click to start a new chat</span>
                    </span>
                <Icon name="bx:bx-plus" class="w-8 h-8"/>
            </button>
        </div>
        <!-- Conversations -->
        <div
            class="flex-1 flex flex-col gap-1 p-3 overflow-y-auto"
        >
            <TransitionGroup name="slide-from-left">
                <div
                    v-for="conversation in sortedConversations"
                    :key="conversation.id"
                    class="flex flex-row items-stretch rounded p-3 pr-0 border-2 border-purple-300/5 hover:border-purple-300/10"
                    :class="{ '!border-purple-300/20': conversation.id === currentConversationId }"
                >
                    <button
                        class="flex flex-col flex-1 text-left overflow-hidden"
                        @click="setCurrentConversationIdHandler(conversation.id)"
                        :disabled="!!processingController"
                        :class="{ 'cursor-not-allowed': !!processingController }"
                    >
                        <span
                            class="text-sm mb-1 flex flex-row items-center justify-center gap-2 w-full"
                            :title="conversation.title"
                        >
                            <GPTIcon
                                v-if="(conversation.activePreset?.client || conversation.activePresetName) === 'bing'"
                                class="h-3 rounded-lg opacity-80"
                            />
                            <GPTIcon
                                v-else-if="(conversation.activePreset?.client || conversation.activePresetName) === 'chatgpt-browser'"
                                class="h-3 text-[#6ea194] rounded-lg opacity-80"
                            />
                            <BingIcon
                                v-else-if="(conversation.activePreset?.client || conversation.activePresetName) === 'bing'"
                                class="h-3 rounded-lg opacity-80"
                            />
                            <ChatIcon
                                v-else-if="(conversation.activePreset?.client || conversation.activePresetName) === 'chat'"
                                class="h-4 rounded-lg opacity-80"
                            />
                            <BarsIcon
                                v-else-if="(conversation.activePreset?.client || conversation.activePresetName) === 'compose'"
                                class="h-4 rounded-lg opacity-80"
                            />
                            <span class="flex-1 truncate">{{ conversation.title || 'New Chat' }}</span>
                        </span>
                        <!-- <span class="text-xs text-white/30 truncate">
                            {{ conversation.id }}
                        </span> -->
                        <span class="text-xs text-white/40">
                            {{ conversation.data.throttling.numUserMessagesInConversation }}/{{ conversation.data.throttling.maxNumUserMessagesInConversation }} - {{ (new Date(conversation.updatedAt)).toLocaleString() }}
                        </span>
                    </button>
                    <button
                        @click="deleteConversationHandler(conversation.id)"
                        class="flex items-center justify-center px-3 text-white/30 hover:text-white/60"
                        :disabled="!!processingController"
                        :class="{ 'cursor-not-allowed': !!processingController }"
                    >
                        <Icon name="bx:bx-trash" class="w-4 h-4"/>
                    </button>
                </div>
            </TransitionGroup>
        </div>
        <!-- Sidebar footer -->
        <div class="flex flex-col items-start gap-2 bg-black/10 p-6 min-h-[100px] text-sm">
            <button
                @click="clearConversationsHandler"
                class="flex gap-1 items-center justify-center text-white/60 transition duration-300 hover:text-white/80"
                :disabled="!!processingController"
                :class="{ 'cursor-not-allowed': !!processingController }"
            >
                <Icon :name="isConfirmingClear ? 'bx:bx-check' : 'bx:bx-trash'" class="w-4 h-4 relative top-px"/>
                Clear all conversations{{ isConfirmingClear ? '?' : '' }}
            </button>
            <div class="flex gap-3 items-center w-full">
                <AppData/>
            </div>
        </div>
    </div>
</template>

<style scoped>
.slide-from-left-move,
.slide-from-left-enter-active,
.slide-from-left-leave-active {
    transition: all 0.3s ease;
}
.slide-from-left-enter-from,
.slide-from-left-leave-to {
    transform: translateX(-320px);
    opacity: 0;
}

.slide-from-left-leave-active {
    opacity: 0;
}
</style>
