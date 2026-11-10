<script lang="ts">
	import GameCanvas from '#lib/components/GameCanvas.svelte';
</script>

<svelte:head>
	<title>BlindLaser</title>
</svelte:head>

<main class="min-h-dvh bg-neutral-950 px-4 py-8 text-neutral-100 sm:px-8">
	<div class="mx-auto flex max-w-4xl flex-col gap-4">
		<header class="flex items-baseline justify-between gap-4">
			<h1 class="text-2xl font-semibold tracking-tight">BlindLaser</h1>
			<p class="text-sm text-neutral-400">Move to aim · Click to fire</p>
		</header>

		<GameCanvas />
	</div>
</main>
