<script setup lang="ts">
import Seg1 from "./segments/1.vue";
import Seg2 from "./segments/2.vue";

const videoRef = ref<HTMLVideoElement | null>(null);

const lastEaseStart = ref<null | number>(null);
const videoDuration = ref(3.753);
const isEasing = computed(() => lastEaseStart.value !== null);

useHead({
	link: [
		{
			rel: "preload",
			as: "image",
			href: "/plaatje-1.png",
			fetchpriority: "high",
		},
		{
			rel: "preload",
			as: "image",
			href: "/plaatje-2.png",
			fetchpriority: "high",
		},
	],
});

const stops = ref<
	{
		time: number;
		component: typeof Seg1;
		label: string;
		visible?: boolean;
	}[]
>([
	{
		time: videoDuration.value,
		component: Seg2,
		label: "0/4",
	},
	{
		time: videoDuration.value * 0.75,
		component: Seg2,
		label: "1/4",
	},
	{
		time: videoDuration.value * 0.5,
		component: Seg2,
		label: "2/4",
	},
	{
		time: videoDuration.value * 0.25,
		component: Seg2,
		label: "3/4",
	},
	{
		time: 0,
		component: Seg1,
		label: "4/4",
	},
]);

onMounted(async () => {
	const video = videoRef.value;
	if (!video) return;

	setComponentVisibility();

	video.addEventListener("loadedmetadata", () => {
		videoDuration.value = video.duration;
	});

	const url = "/drone-short.mp4?t=" + Date.now();

	try {
		const response = await fetch(url);
		const blob = await response.blob();
		const objectURL = URL.createObjectURL(blob);

		video.src = objectURL;

		video.load();
	} catch (error) {
		console.error("Error loading video:", error);
	}
});

function ParametricBlend(t: number): number {
	const sqr = t * t;
	return sqr / (2.0 * (sqr - t) + 1.0);
}

async function easeTo(to: number) {
	const video = videoRef.value;
	if (!video) return;

	const from = video.currentTime;
	const duration = 1e3;

	lastEaseStart.value = Date.now();
	let latest = lastEaseStart.value;

	const start = Date.now();

	const ease = async () => {
		const t = Date.now();
		const progress = (t - start) / duration;

		video.currentTime = from + (to - from) * Math.min(progress, 1);

		const blend = ParametricBlend(progress);
		video.currentTime = from + (to - from) * Math.min(blend, 1);
		await video.play();
		video.pause();

		if (progress < 1 && latest === lastEaseStart.value) {
			requestAnimationFrame(ease);
		} else if (progress >= 1) {
			lastEaseStart.value = null;
			setComponentVisibility();
		}
	};

	requestAnimationFrame(ease);
}

function setComponentVisibility() {
	for (const stop of stops.value) stop.visible = false;

	const video = videoRef.value;
	if (!video) return;

	const currentTime = video.currentTime;
	const closestStops = [...stops.value].sort((a, b) => {
		return Math.abs(a.time - currentTime) - Math.abs(b.time - currentTime);
	});

	const distance = Math.abs(closestStops[0].time - currentTime);
	if (distance < 0.1) closestStops[0].visible = true;
}
</script>

<template>
	<div>
		<div class="w-full h-screen grid grid-cols-1">
			<video
				muted
				ref="videoRef"
				playsinline
				webkit-playsinline
				class="object-cover relative h-full -z-1"
			/>
		</div>
		<component
			v-for="stop in stops"
			:key="stop.label"
			:is="stop.component"
			class="transition-opacity duration-100"
			:class="stop.visible && !isEasing ? 'opacity-100' : 'opacity-0'"
		/>
		<div
			class="flex justify-between absolute z-20 bottom-12 left-1/2 -translate-x-1/2 bg-white px-12 rounded-full"
		>
			<button
				v-for="stop of stops"
				:key="stop.label"
				@click="easeTo(stop.time)"
			>
				{{ stop.label }}
			</button>
		</div>
	</div>
</template>

<style scoped>
button {
	@apply px-2 py-3;
}
</style>
