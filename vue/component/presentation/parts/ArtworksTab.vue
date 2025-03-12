<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps({
    spaceItems: {
        type: Array,
        required: true,
    }
});

const emit = defineEmits(['update:currentSpaceItem']);

const selectSpaceItem = (index) => {
    emit('update:currentSpaceItem', props.spaceItems[index]);
    console.log(props.spaceItems[index]);
};

const openPopupArtView = (item) => {
    console.log('Opening artwork popup', item);
};
</script>

<template>
    <div class="real-product-items">
        <div
            v-for="(item, index) in spaceItems"
            :key="item.id"
            class="real-product-items__item"
        >
            <div
                class="real-product-items__item_image"
                v-html="item.product.imageBlock"
                @click="selectSpaceItem(index)"
            ></div>
            <div class="real-product-items__item_name">{{ item.product.name }}</div>
            <div class="real-product-items__item_author">
                {{ item.product.s_firstname }} {{ item.product.s_lastname }}
            </div>
        </div>
    </div>
</template>
