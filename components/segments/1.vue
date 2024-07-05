<script setup lang="ts">
import Image from "./Image.vue";
</script>

<template>
	<div class="absolute top-0 left-0 w-full h-full">
		<div
			class="absolute top-0 left-0 w-full h-full z-10 flex justify-center items-center"
		>
			<div class="max-w-xl bg-white p-5 rounded-lg">
				<h2 class="text-black font-bold text-2xl">Hoi!</h2>
				<p class="mt-3">
					Dit is een component. Ik kan hier tooltips, plaatjes, etc. In stoppen.
					De achtergrond die je nu ziet is een plaatje, niet het filmpje.
				</p>
			</div>
		</div>
		<Image src="/plaatje-1.png" />
	</div>
</template>
