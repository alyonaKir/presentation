<script setup>
import { ref } from 'vue';
import { defineModel } from 'vue';
import ArtworkTab from './ArtworksTab.vue';
import CommentsTab from './CommentsTab.vue';

const props = defineProps({
    selectedSpaceName: {
        type: String,
        required: true,
    },
    currentSpaceItem: {
        type: Object,
        required: true,
    },
    currentCustomerData: {
        type: Object,
        required: true,
    },
    spaceItems: {
        type: Array,
        required: true,
    },
});

const emit = defineEmits(['showArtworkTab', 'showComments', 'update:currentSpaceItem']);
const showArtViewings = ref(false);
const showComments = ref(false);
const isSidebarVisible = ref(true);
const isPlayScreenVisible = defineModel('isPlayScreenVisible', { type: Boolean, default: true });

const toggleSidebar = () => {
    isSidebarVisible.value = !isSidebarVisible.value;
};
const playMode = () => {
    isPlayScreenVisible.value = true;
};

const openArtworksTab = () => {
    showArtViewings.value = !showArtViewings.value;
    emit('showArtworkTab');
};
const openComments = () => {
    showComments.value = !showComments.value;
    emit('showComments');
};
</script>

<template>
    <div class="sidebar blur" v-show="isPlayScreenVisible">
        <button v-show="!isSidebarVisible" @click="toggleSidebar">Close</button>
        <button class="realPresentation-sidebar burger" @click="toggleSidebar"></button>
        <button class="realPresentation-sidebar play" @click="playMode"></button>
        <button class="realPresentation-sidebar spaces"></button>
        <button class="realPresentation-sidebar artworks" @click="openArtworksTab"></button>
        <button class="realPresentation-sidebar comments" @click="openComments"></button>
        <button class="realPresentation-sidebar documents"></button>
        <button class="realPresentation-sidebar cart"></button>
        <div class="bottom-panel">
            <div class="avatar"></div>
            <button class="realPresentation-sidebar__exit"></button>
        </div>
        <div class="tab" v-if="showArtViewings">
            <ArtworkTab
                v-if="showArtViewings"
                :spaceItems="spaceItems"
                @update:currentSpaceItem="$emit('update:currentSpaceItem', $event)"
            />
        </div>
        <div class="tab" v-if="showComments">
            <CommentsTab
                v-if="showComments"
                :currentSpaceItem = "currentSpaceItem"
            />
        </div>
    </div>



</template>
