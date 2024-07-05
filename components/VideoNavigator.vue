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

function ParametricBlend(t: number): number {
	const sqr = t * t;
	return sqr / (2.0 * (sqr - t) + 1.0);
}

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

		const blend = ParametricBlend(progress);
		video.currentTime = from + (to - from) * Math.min(blend, 1);
		await video.play();
		video.pause();

		if (progress < 1 && latest === lastEaseStart) {
			requestAnimationFrame(ease);
		}
	};

	requestAnimationFrame(ease);
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
		<div
			class="flex justify-between absolute bottom-12 left-1/2 -translate-x-1/2 bg-white px-12 rounded-full"
		>
			<button @click="easeTo(0)">0/4</button>
			<button @click="easeTo(videoDuration * 0.25)">1/4</button>
			<button @click="easeTo(videoDuration * 0.5)">2/4</button>
			<button @click="easeTo(videoDuration * 0.75)">3/4</button>
			<button @click="easeTo(videoDuration * 1)">4/4</button>
		</div>
	</div>
</template>

<style scoped>
button {
	@apply px-2 py-3;
}
</style>
