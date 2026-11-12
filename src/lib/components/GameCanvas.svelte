<script lang="ts">
	import { onMount } from 'svelte';

	// Fixed world coordinate space. All game logic uses these units.
	const WORLD_W = 1000;
	const WORLD_H = 1000;

	let player = { x: WORLD_W / 2, y: WORLD_H / 2 };
	let aim = { x: WORLD_W * 0.75, y: WORLD_H * 0.35 };

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let raf = 0;
		let alive = true;

		// Keep the bitmap resolution in sync with its CSS size and DPR.
		function resize() {
			const dpr = Math.min(window.devicePixelRatio, 2);
			const rect = canvas.getBoundingClientRect();
			canvas.width = Math.max(1, Math.round(rect.width * dpr));
			canvas.height = Math.max(1, Math.round(rect.height * dpr));
		}

		const observer = new ResizeObserver(resize);
		observer.observe(canvas);
		resize();

		// Screen coordinates -> world coordinates.
		function toWorld(e: PointerEvent) {
			const rect = canvas.getBoundingClientRect();
			return {
				x: ((e.clientX - rect.left) / rect.width) * WORLD_W,
				y: ((e.clientY - rect.top) / rect.height) * WORLD_H
			};
		}

		function onPointerMove(e: PointerEvent) {
			aim = toWorld(e);
		}

		function onPointerDown(e: PointerEvent) {
			aim = toWorld(e);
			player = { ...aim };
		}

		canvas.addEventListener('pointermove', onPointerMove);
		canvas.addEventListener('pointerdown', onPointerDown);

		function draw() {
			// Map world units onto the bitmap.
			ctx.setTransform(canvas.width / WORLD_W, 0, 0, canvas.height / WORLD_H, 0, 0);

			// Background
			ctx.fillStyle = '#05070c';
			ctx.fillRect(0, 0, WORLD_W, WORLD_H);

			// Player
			ctx.save();
			ctx.fillStyle = '#e0f2fe';
			ctx.beginPath();
			ctx.arc(player.x, player.y, 20, 0, Math.PI * 2);
			ctx.fill();
			ctx.restore();

			// Crosshair
			ctx.save();
			ctx.beginPath();
			ctx.strokeStyle = '#e0f2feb3';
			ctx.lineWidth = 1.5;
			ctx.arc(aim.x, aim.y, 8, 0, Math.PI * 2);
			ctx.moveTo(aim.x - 14, aim.y);
			ctx.lineTo(aim.x - 4, aim.y);
			ctx.moveTo(aim.x + 4, aim.y);
			ctx.lineTo(aim.x + 14, aim.y);
			ctx.moveTo(aim.x, aim.y - 14);
			ctx.lineTo(aim.x, aim.y - 4);
			ctx.moveTo(aim.x, aim.y + 4);
			ctx.lineTo(aim.x, aim.y + 14);
			ctx.stroke();
			ctx.restore();

			// Arrow
			const dx = aim.x - player.x;
			const dy = aim.y - player.y;
			const angle = Math.atan2(dy, dx);

			const DISTANCE = 80; 

			const x = player.x + Math.cos(angle) * DISTANCE;
			const y = player.y + Math.sin(angle) * DISTANCE;

			ctx.save();
			ctx.translate(x, y);
			ctx.rotate(angle); 
			ctx.font = '3em sans-serif';
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';
			ctx.fillStyle = '#e0f2fe';
			ctx.fillText('➤', 0, 0);
			ctx.restore();
		}

		function frame() {
			if (!alive) return;
			draw();
			raf = requestAnimationFrame(frame);
		}
		raf = requestAnimationFrame(frame);

		return () => {
			alive = false;
			cancelAnimationFrame(raf);
			observer.disconnect();
			canvas.removeEventListener('pointermove', onPointerMove);
			canvas.removeEventListener('pointerdown', onPointerDown);
		};
	});
</script>

<canvas
	bind:this={canvas}
	class="block aspect-square w-full touch-none rounded-xl border border-neutral-800 bg-black"
></canvas>
