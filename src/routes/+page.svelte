<script lang="ts">
	import ArcDisplay from '../lib/components/ArcDisplay.svelte';

	let cpuUsage = 0;
	let cpuTemp = 0;
	let gpuUsage = 0;
	let gpuTemp = 0;
	let ramUsage = 0;
	let gpuRamUsage = 0;

	const interval = setInterval(() => {
		cpuUsage = Math.random() * 10 + 35;
		cpuTemp = Math.random() * 10 + 47;
		gpuUsage = Math.random() * 10 + 88;
		gpuTemp = Math.random() * 10 + 79;
		ramUsage = Math.random() * 10 + 59;
		gpuRamUsage = Math.random() * 10 + 79;
	}, 1000);

	$: {
		() => {
			clearInterval(interval);
		};
	}
</script>

<div class="grid">
	<div class="cpu">
		<div class="big">
			<ArcDisplay
				currentValue={cpuUsage}
				maxValue={89}
				component="CPU"
				size={480}
				description="Ryzen 7 7800X3D"
			/>
		</div>
		<div class="small">
			<ArcDisplay
				currentValue={cpuTemp}
				maxValue={100}
				component="CPU"
				size={(640 / 6) * 2}
				description="TEMP"
			/>
		</div>
		<div class="small">
			<ArcDisplay
				currentValue={ramUsage}
				maxValue={100}
				component="RAM"
				size={(640 / 6) * 2}
				description="USAGE"
			/>
		</div>
	</div>

	<div class="gpu">
		<div class="big">
			<ArcDisplay
				currentValue={gpuUsage}
				maxValue={105}
				component="GPU"
				size={480}
				description="GeForce RTX 4090"
			/>
		</div>
		<div class="small">
			<ArcDisplay
				currentValue={gpuTemp}
				maxValue={100}
				component="GPU"
				size={(640 / 6) * 2}
				description="TEMP"
			/>
		</div>
		<div class="small">
			<ArcDisplay
				currentValue={gpuRamUsage}
				maxValue={100}
				component="RAM"
				size={(640 / 6) * 2}
				description="USAGE"
			/>
		</div>
	</div>
</div>

<style>
	.grid {
		--padding-top: 2.5rem;
		--padding-bottom: 1rem;
		--width: 1280px;
		--height: calc(800px - var(--padding-top) - var(--padding-bottom));

		display: grid;
		grid-template-columns: repeat(16, calc(var(--width) / 16));
		grid-template-rows: repeat(9, calc(var(--height) / 9));
		width: var(--width);
		height: var(--height);
		padding-top: var(--padding-top);
		padding-bottom: var(--padding-bottom);
		margin: auto auto;
		background-color: #222;
	}

	.cpu,
	.gpu {
		position: relative;
		display: grid;
		grid-template-columns: repeat(6, 1fr);
		grid-template-rows: repeat(6, 1fr);
		grid-column: span 8;
		grid-row: span 9;
	}

	.small,
	.big {
		align-self: center;
		justify-self: center;
	}

	.big {
		grid-column: 1 / span 6;
		grid-row: 1 / span 4;
	}

	.small {
		grid-column: span 3;
		grid-row: 5 / span 3;
	}
</style>
