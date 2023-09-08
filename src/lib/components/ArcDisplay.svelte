<script lang="ts">
	import { browser } from '$app/environment';

	export let currentValue: number;
	export let maxValue: number;
	export let component: string;
	export let description: string;
	export let size: number = 320;

	let percentage: number;

	$: {
		percentage = currentValue <= maxValue ? currentValue / maxValue : 1;
	}

	// Arc construction
	const describeArc = (
		x: number,
		y: number,
		radius: number,
		startPercentage: number,
		endPercentage: number
	) => {
		const maxAngle = 270;
		const startAngle = maxAngle * startPercentage;
		const endAngle = maxAngle * endPercentage;

		const polarToCartesian = (
			centerX: number,
			centerY: number,
			radius: number,
			angleInDegrees: number
		) => {
			const angleInRadians = ((angleInDegrees + 135) * Math.PI) / 180.0;
			return {
				x: centerX + radius * Math.cos(angleInRadians),
				y: centerY + radius * Math.sin(angleInRadians)
			};
		};

		const start = polarToCartesian(x, y, radius, endAngle);
		const end = polarToCartesian(x, y, radius, startAngle);
		const largeArcFlag = endAngle - startAngle <= 180 ? '0' : '1';

		return `M ${start.x} ${start.y} A ${radius} ${radius} 0 ${largeArcFlag} 0 ${end.x} ${end.y}`;
	};

	// Color blending
	function blendColors(color1: number[], color2: number[], percentage: number) {
		const [r1, g1, b1] = color1;
		const [r2, g2, b2] = color2;
		const r = Math.round(r1 + (r2 - r1) * percentage);
		const g = Math.round(g1 + (g2 - g1) * percentage);
		const b = Math.round(b1 + (b2 - b1) * percentage);

		return `rgb(${r}, ${g}, ${b})`;
	}

	const color1 = [0, 0, 255];
	const color2 = [120, 0, 255];
	const color3 = [255, 0, 50];
	let newColor: string;

	$: {
		if (percentage < 0.75) {
			newColor = blendColors(color1, color2, percentage / 0.75);
		} else if (percentage < 0.9) {
			newColor = blendColors(color2, color3, (percentage - 0.75) / (0.9 - 0.75));
		} else {
			newColor = `rgb(${color3[0]}, ${color3[1]}, ${color3[2]})`;
		}
	}

	// Smooth color transition
	let animatingColor: string = '';
	let targetColor: string = '';
	let colorFrameId: number;

	function parseRGB(colorString: string) {
		const match = colorString.match(/(\d+),\s*(\d+),\s*(\d+)/);
		return match ? [parseInt(match[1]), parseInt(match[2]), parseInt(match[3])] : [0, 0, 0];
	}

	$: if (browser) {
		if (colorFrameId) {
			cancelAnimationFrame(colorFrameId);
		}

		const initialColorArray = parseRGB(animatingColor || newColor);
		const targetColorArray = parseRGB(newColor);

		const startTime = performance.now();

		const animateColor = (time: number) => {
			const elapsed = time - startTime;
			const duration = 300; // 16.67 ms

			if (elapsed < duration) {
				const percentage = elapsed / duration;
				animatingColor = blendColors(initialColorArray, targetColorArray, percentage);
				colorFrameId = requestAnimationFrame(animateColor);
			} else {
				animatingColor = targetColor;
			}
		};

		colorFrameId = requestAnimationFrame(animateColor);
	}

	// Smooth value transition
	let animationFrameId: number;
	let animatingPercentage: number = 0;

	$: if (browser) {
		if (animationFrameId) {
			cancelAnimationFrame(animationFrameId);
		}

		const startTime = performance.now();
		const initialPercentage = animatingPercentage;

		const animate = (time: number) => {
			const elapsed = time - startTime;
			const duration = 2000; // 1000 ms

			if (elapsed < duration) {
				animatingPercentage =
					initialPercentage + (percentage - initialPercentage) * (elapsed / duration);
				animationFrameId = requestAnimationFrame(animate);
			} else {
				animatingPercentage = percentage;
			}
		};

		animationFrameId = requestAnimationFrame(animate);
	}

	$: displayedPercentage = Math.round(animatingPercentage * 100);

	const styleString = `--size: ${size}px`;
</script>

<div class="gauge" style={styleString}>
	<svg width="100%" height="100%" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 300 300">
		<path
			id="ArcBackground"
			fill="none"
			stroke="#404040"
			stroke-width="20"
			d={describeArc(150, 150, 130, 0, 1)}
		/>
		<path
			id="UsageArc"
			fill="none"
			stroke={animatingColor}
			stroke-width="40"
			d={describeArc(150, 150, 130, 0, animatingPercentage)}
		/>
	</svg>

	<div class="text">
		<p class="value">{displayedPercentage}</p>
		<div class="component">
			<p class="component-value">{component}</p>
			<p class="component-description">{description}</p>
		</div>
	</div>
</div>

<style>
	.gauge {
		position: relative;
		width: var(--size);
		height: var(--size);
	}

	.text {
		position: absolute;
		top: 0;
		font-family: 'Gilroy-Bold';
		text-align: center;
		color: white;
	}

	.value {
		margin: 0;
		width: var(--size);
		height: var(--size);
		font-size: calc(var(--size) * 0.384);
		line-height: var(--size);
	}

	.component {
		position: absolute;
		bottom: calc(var(--size) * 0.03);
		margin: 0;
		width: var(--size);
		font-size: calc(var(--size) * 0.128);
	}

	.component-value,
	.component-description {
		margin: 0;
	}

	.component-description {
		font-size: calc(var(--size) * 0.064);
	}
</style>
