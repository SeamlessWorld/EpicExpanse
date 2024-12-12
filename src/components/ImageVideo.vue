<template>
    <section>
        <h3>{{ title }}</h3>
        <div class="panel">
            <div ref="carouselElement" :id="id" class="relative" data-twe-carousel-init data-twe-carousel-slide
                data-twe-ride="carousel" data-twe-interval="9999999">

                <div class="relative w-full overflow-hidden after:clear-both after:block after:content-['']">
                    <!-- Carousel Items -->
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4" data-twe-carousel-item>
                        <div v-for="(item, i) in items" :key="i" class="flex flex-col items-center">
                            <div class="flex flex-col md:flex-row items-center">
                                <!-- Left Image -->
                                <div class="image-container w-full md:w-1/2 flex justify-center">
                                    <img :src="item.image" alt="Image Description"
                                        class="rounded-lg w-full h-auto object-cover">
                                </div>

                                <!-- Right Video -->
                                <div class="video-container w-full md:w-1/2 flex justify-center">
                                    <video :ref="(el: any) => videos[i] = el" v-lazy controls :src="item.video"
                                        class="rounded-lg"></video>
                                </div>
                            </div>

                            <!-- Prompt/Description -->
                            <div class="prompt-container mt-4 text-center">
                                <p class="text-sm font-light leading-relaxed">{{ item.prompt }}</p>
                            </div>
                        </div>
                    </div>
                </div>


                <!--Carousel indicators-->
                <div v-if="items.length > 1"
                    class="absolute bottom-0 left-0 right-0 z-[2] mx-[15%] -mb-8 flex list-none justify-center p-0"
                    data-twe-carousel-indicators>
                    <button v-for="(_item, i) in items" :key="i" :ref="(el: any) => carouselIndicators[i] = el"
                        type="button" :data-twe-target="`#${id}`" :data-twe-slide-to="i" class="indicator"
                        aria-current="true" :aria-label="`Slide ${i + 1}`"></button>
                </div>

                <!--Carousel controls - prev item-->
                <button v-if="items.length > 1" class="indicator-btn indicator-left-btn" type="button"
                    :data-twe-target="`#${id}`" data-twe-slide="prev">
                    <span class="inline-block h-8 w-8">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                            stroke="currentColor" class="h-6 w-6">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
                        </svg>
                    </span>
                    <span
                        class="!absolute !-m-px !h-px !w-px !overflow-hidden !whitespace-nowrap !border-0 !p-0 ![clip:rect(0,0,0,0)]">Previous</span>
                </button>
                <!--Carousel controls - next item-->
                <button v-if="items.length > 1" class="indicator-btn indicator-right-btn" type="button"
                    :data-twe-target="`#${id}`" data-twe-slide="next">
                    <span class="inline-block h-8 w-8">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                            stroke="currentColor" class="h-6 w-6">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
                        </svg>
                    </span>
                    <span
                        class="!absolute !-m-px !h-px !w-px !overflow-hidden !whitespace-nowrap !border-0 !p-0 ![clip:rect(0,0,0,0)]">Next</span>
                </button>
            </div>
        </div>
    </section>

</template>

<script setup lang="ts">
interface Props {
    id?: string
    title?: string
    items?: { image: string; video: string; prompt: string }[]
}
const { props } = defineProps<{ props: Props }>()
const title = props.title || ''
const items = (props.items || []).map(item => ({
    image: item.image.startsWith("assets") ? new URL(`../${item.image}`, import.meta.url).href : item.image,
    video: item.video.startsWith("assets") ? new URL(`../${item.video}`, import.meta.url).href : item.video,
    prompt: item.prompt,
}))
const id = props.id || title.replaceAll(" ", "")

import { ref, onMounted } from 'vue';
import { initTWE, Carousel } from 'tw-elements';
import { store } from '@/store'

const carouselElement = ref<HTMLElement>();
const videos = ref<HTMLVideoElement[]>([]);
const carouselItems = ref<HTMLElement[]>([]);
const carouselIndicators = ref<HTMLElement[]>([]);

onMounted(async () => {
    carouselItems.value[0]?.setAttribute("data-twe-carousel-active", "")
    carouselIndicators.value[0]?.setAttribute("data-twe-carousel-active", "")
    do {
        await new Promise(resolve => setTimeout(resolve, 100))
    } while (store.tweInitializing["Carousel"])
    store.setInitializing("Carousel", true)
    console.log("initializing..", store.tweInitializing["Carousel"])
    initTWE({ Carousel }, { allowReinits: true, checkOtherImports: true });
    store.setInitializing("Carousel", false)
    console.log("initialized", store.tweInitializing["Carousel"])

    carouselElement.value?.addEventListener('slide.twe.carousel', (v: any) => {
        const from = v.from;
        const to = v.to;
        videos.value[from]?.pause();
        // if (inVisible(videos.value[from])) {
        //     videos.value[to].play();
        // }

    })
});

</script>

<style scoped lang="scss">
section {
    @apply w-full py-10 md:px-16 px-6;
    @apply flex flex-col justify-center items-center;
}

.panel {
    max-width: 960px;
    @apply w-full mt-2;

    &>* {
        @apply w-full mb-8;
    }

    :last-child {
        @apply mb-0;
    }
}

.video-group {
    @apply flex flex-col md:flex-row items-center justify-between gap-4;
}

.grid {
    @apply w-full gap-4;
    /* Space between items */
}

.image-container,
.video-container {
    @apply flex justify-center items-center;
}

img,
video {
    @apply w-full h-full object-cover rounded-lg;
    /* Ensure equal size for image and video */
}

.prompt-container {
    @apply mt-4 text-center flex justify-center;

    p {
        @apply text-sm font-light leading-relaxed;
    }
}

.carousel-indicators {
    @apply flex list-none justify-center p-0;
}

.indicator-btn {
    @apply inline-block h-8 w-8 rounded-full bg-gray-200 hover:bg-gray-400;
}

.indicator {
    @apply w-3 h-3 mx-1 bg-gray-300 rounded-full;

    &[aria-current="true"] {
        @apply bg-gray-600;
    }
}

.t2i-caption {
    @apply font-light italic md:px-20 text-center leading-snug;
}

video {
    @apply rounded-lg
}

.item-content {
    @apply flex flex-col-reverse md:flex-row md:justify-between md:items-center;

    video {
        @apply w-full md:w-1/2
    }

    div {
        @apply md:w-1/2 md:pl-2 md:pr-20 md:mt-0;
        @apply w-full mx-2 mt-2;
        @apply font-thin leading-tight;
        text-align: justify;
    }
}
</style>
