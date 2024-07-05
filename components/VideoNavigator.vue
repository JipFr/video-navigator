<script setup lang="ts">
const videoRef = ref<HTMLVideoElement | null>(null);

let lastEaseStart: null | number = null;
const videoDuration = ref(0);

onMounted(async () => {
	const video = videoRef.value;
	if (!video) return;

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

async function easeTo(to: number) {
	const video = videoRef.value;
	if (!video) return;

	const from = video.currentTime;
	const duration = 1e3;

	lastEaseStart = Date.now();
	let latest = lastEaseStart;

	const start = Date.now();

	const ease = async () => {
		const t = Date.now();
		const progress = (t - start) / duration;

		video.currentTime = from + (to - from) * Math.min(progress, 1);
		console.log(video.currentTime);
		console.time("play");
		await video.play();
		console.timeEnd("play");

		if (progress < 1 && latest === lastEaseStart) {
			requestAnimationFrame(ease);
		}
	};

	requestAnimationFrame(ease);
}

function randomiseEase() {
	const video = videoRef.value;
	if (!video) return;

	const to = Math.random() * video.duration;
	console.log("Randomising to", to);
	easeTo(to);
}
</script>

<template>
	<div class="w-full h-full grid grid-cols-1">
		<video
			controls
			muted
			ref="videoRef"
			playsinline
			webkit-playsinline
			class="w-full h-screen object-cover"
		/>
		{{ videoDuration }}
		<button @click="easeTo(0)">0/4</button>
		<button @click="easeTo(videoDuration * 0.25)">1/4</button>
		<button @click="easeTo(videoDuration * 0.5)">2/4</button>
		<button @click="easeTo(videoDuration * 0.75)">3/4</button>
		<button @click="easeTo(videoDuration * 1)">4/4</button>
		<button class="my-12" @click="randomiseEase()">Randomise with ease</button>
	</div>
</template>
