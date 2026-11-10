<script lang="ts">
	import { onMount } from 'svelte';

	// Fixed "world" coordinate space. All game logic uses these units.
	const WORLD_W = 960;
	const WORLD_H = 600;

	// Change these from your game logic whenever you like.
	let player = { x: WORLD_W / 2, y: WORLD_H / 2 };
	let aim = { x: WORLD_W * 0.75, y: WORLD_H * 0.35 };

	type Beam = { x1: number; y1: number; x2: number; y2: number; life: number };
	let beams: Beam[] = [];

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let raf = 0;
		let alive = true;
		let last = performance.now();

		// Keep the bitmap resolution in sync with its CSS size (and device pixel ratio).
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
			beams.push({
				x1: player.x,
				y1: player.y,
				x2: aim.x,
				y2: aim.y,
				life: 1
			});
			if (beams.length > 8) beams.shift();
		}

		canvas.addEventListener('pointermove', onPointerMove);
		canvas.addEventListener('pointerdown', onPointerDown);

		function draw(dt: number) {
			// Map world units onto the bitmap.
			ctx.setTransform(canvas.width / WORLD_W, 0, 0, canvas.height / WORLD_H, 0, 0);
			ctx.clearRect(0, 0, WORLD_W, WORLD_H);

			// Background
			ctx.fillStyle = '#05070c';
			ctx.fillRect(0, 0, WORLD_W, WORLD_H);

			// Grid (helps you see the coordinate space)
			ctx.strokeStyle = 'rgba(148, 163, 184, 0.10)';
			ctx.lineWidth = 1;
			ctx.beginPath();
			for (let x = 0; x <= WORLD_W; x += 40) {
				ctx.moveTo(x, 0);
				ctx.lineTo(x, WORLD_H);
			}
			for (let y = 0; y <= WORLD_H; y += 40) {
				ctx.moveTo(0, y);
				ctx.lineTo(WORLD_W, y);
			}
			ctx.stroke();

			// Age out old beams
			for (const b of beams) b.life -= dt * 0.7;
			beams = beams.filter((b) => b.life > 0);

			// Faint aim line
			ctx.save();
			ctx.setLineDash([6, 8]);
			ctx.strokeStyle = 'rgba(34, 211, 238, 0.35)';
			ctx.lineWidth = 2;
			ctx.beginPath();
			ctx.moveTo(player.x, player.y);
			ctx.lineTo(aim.x, aim.y);
			ctx.stroke();
			ctx.restore();

			// Lasers
			ctx.save();
			ctx.lineCap = 'round';
			for (const b of beams) {
				ctx.globalAlpha = Math.max(0, b.life);

				// Outer glow
				ctx.shadowColor = '#22d3ee';
				ctx.shadowBlur = 16;
				ctx.strokeStyle = '#22d3ee';
				ctx.lineWidth = 6;
				ctx.beginPath();
				ctx.moveTo(b.x1, b.y1);
				ctx.lineTo(b.x2, b.y2);
				ctx.stroke();

				// Bright core
				ctx.shadowBlur = 0;
				ctx.strokeStyle = '#e0f2fe';
				ctx.lineWidth = 2;
				ctx.beginPath();
				ctx.moveTo(b.x1, b.y1);
				ctx.lineTo(b.x2, b.y2);
				ctx.stroke();
			}
			ctx.restore();

			// Player circle
			ctx.save();
			ctx.shadowColor = '#67e8f9';
			ctx.shadowBlur = 24;
			ctx.fillStyle = '#e0f2fe';
			ctx.beginPath();
			ctx.arc(player.x, player.y, 16, 0, Math.PI * 2);
			ctx.fill();
			ctx.restore();

			// Crosshair at the aim point
			ctx.save();
			ctx.strokeStyle = 'rgba(224, 242, 254, 0.7)';
			ctx.lineWidth = 1.5;
			ctx.beginPath();
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
		}

		function frame(now: number) {
			if (!alive) return;
			const dt = Math.min((now - last) / 1000, 0.05);
			last = now;
			draw(dt);
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
	class="block aspect-[8/5] w-full touch-none rounded-xl border border-neutral-800 bg-black"
></canvas>
