<template>
    <div class="realPresentation-image" :style="{ backgroundImage: getBgImageUrl }">
        <div id="image-onload-error" v-if="imageLoadError">Could not locate image.</div>
        <div class="space-list dev blur" v-show="isPlayScreenAvailable">
            <span class="title dev" @click="toggleDropdown">
                {{ selectedSpaceName }} <span class="icon-drop"></span>
            </span>
            <ul class="list dev" v-show="isDropdownOpen">
                <li v-for="space in spaceArr" :key="space.id">
                    <a href="#" @click.prevent="selectSpace(space)">{{ space.name }}</a>
                </li>
            </ul>
        </div>
    </div>
    <p>Play Screen Visible: {{ isPlayScreenAvailable }}</p>
    <Sidebar
        v-model:isPlayScreenVisible="isPlayScreenAvailable"
        :selectedSpaceName="selectedSpaceName"
        :currentSpaceItem="currentSpaceItem"
        :currentCustomerData="currentCustomerData"
        :spaceItems="spaceItems"
    />

    <ArtworkCard
        v-show="isPlayScreenAvailable"
        :spaceItems="spaceItems"
        :isPurchaseClient="defaults.purchaseBy === 'client'"
        :isMultipleProject="defaults.multipleProject"
        :currentSpaceItem="currentSpaceItem"
        @update:currentSpaceItem="updateCurrentSpaceItem"
    />
</template>

<script>
import { ref } from 'vue';
import Sidebar from "../parts/Sidebar.vue";
import ArtworkCard from "../parts/ArtworkCard.vue";

export default {
    components: { ArtworkCard, Sidebar },
    data() {
        return {
            isPlayScreenAvailable: true,
            imagePath: '',
            spaceArr: [],
            selectedSpaceName: 'Select Space',
            currentSpaceItem: null,
            currentCustomerData: null,
            defaults: {},
            isDropdownOpen: false,
            spaceItems: [],
            imageLoadError: false,
        };
    },
    computed: {
        getBgImageUrl() {
            return `url('${this.imagePath}')`;
        },
    },
    methods: {
        async fetchCustomerData() {
            try {
                const response = await fetch('/rest/V1/customers/me');
                this.currentCustomerData = await response.json();
            } catch (error) {
                console.error('Failed to fetch current user information.');
            }
        },
        loadImage() {
            const img = new Image();
            img.src = this.imagePath;
            img.onload = () => { this.imageLoadError = false; };
            img.onerror = () => { this.imageLoadError = true; };
        },
        selectSpace(space) {
            this.selectedSpaceName = this.truncateText(space.name, 18);
            this.isDropdownOpen = false;
            this.updateSpaceItems(space);
        },
        truncateText(text, maxLength) {
            return text.length > maxLength ? text.substring(0, maxLength) + '...' : text;
        },
        updateSpaceItems(space) {
            if (space.items.length) {
                this.spaceItems = space.items;
                this.setSpaceItem(space.items[0], space);
            }
        },
        setSpaceItem(selectedItem, space) {
            this.currentSpaceItem = selectedItem;
            this.updateImagePath(selectedItem);
        },
        updateImagePath(selectedItem) {
            this.imagePath = selectedItem.options?.s3_path
                ? `https://s3.amazonaws.com/static.ginnava.com/${selectedItem.options.s3_path}`
                : selectedItem.product.image_url;
            this.loadImage();
        },
        toggleDropdown() {
            this.isDropdownOpen = !this.isDropdownOpen;
        },
        initializeImagePath() {
            if (!Array.isArray(this.spaceArr)) return;

            const preselectedSpace = this.spaceArr.find(space =>
                this.defaults?.preselectedSpaceId === space.id);

            const spaceToLoad = preselectedSpace || this.spaceArr.find(space => space.items.length);
            if (spaceToLoad) this.selectSpace(spaceToLoad);

            const params = new URLSearchParams(window.location.search);
            const spaceId = params.get('space-id');
            if (spaceId) {
                const selectedSpace = this.spaceArr.find(space => space.id == spaceId);
                if (selectedSpace) this.selectSpace(selectedSpace);
            }
        },
        updateCurrentSpaceItem(newValue) {
            const selectedSpace = this.spaceArr.find(space => space.items.includes(newValue));
            if (selectedSpace) this.setSpaceItem(newValue, selectedSpace);
        },
    },
    mounted() {
        this.fetchCustomerData();
        const element = document.getElementById('realPresentation-dev');
        if (element) {
            try {
                this.defaults = JSON.parse(JSON.parse(element.getAttribute('data-defaults')));
            } catch (error) {
                console.error('Error parsing JSON:', error);
            }
        }
        this.spaceArr = this.defaults.spacesData?.spaces || [];
        this.initializeImagePath();
    },
    watch: {
        currentSpaceItem(newValue) {
            console.log('Space item changed:', newValue);
            this.updateCurrentSpaceItem(newValue);
        },
    },
};
</script>
