<script lang="ts">
	import GameCanvas from '#lib/components/GameCanvas.svelte';
</script>

<svelte:head>
	<title>BlindLaser</title>
</svelte:head>
<main
	class="grid h-dvh w-full place-items-center overflow-hidden bg-neutral-950 p-2 text-neutral-100"
>
	<div class="relative aspect-square w-[min(100%,calc(100dvh-1rem))]">
		<GameCanvas />
	</div>
</main>
