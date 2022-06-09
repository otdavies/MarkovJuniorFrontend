<script lang="ts">
	import { onMount } from 'svelte';
	import Two from 'two.js';
	import type { Rectangle } from 'two.js/src/shapes/rectangle';
	import Color from './Color.svelte';
	import Panel from './Panel.svelte';

	let worksurface: HTMLElement;
	let renderer: Two;

	let gridUnit: number = 24;

	onMount(async () => {
		renderer = initializeRenderer();
		let a = drawPixel(5, 5, new PixelColor(255, 0, 0));
		let b = drawPixel(3, 7, new PixelColor(255, 0, 0));
		let c = drawPixel(2, 7, new PixelColor(255, 255, 255));
		let d = drawPixel(6, 5, new PixelColor(255, 255, 255));
		drawConnection(a, b, new PixelColor(255, 0, 0));
		renderer.update();
	});

	function onMouseMove(event: any) {
		console.log(event);
	}

	class PixelColor {
		rgb: string;
		constructor(r: number, g: number, b: number) {
			this.rgb = 'rgb(' + r + ', ' + g + ', ' + b + ')';
		}
	}

	function drawPixel(x: number, y: number, color: PixelColor) {
		x = x * gridUnit + 12;
		y = y * gridUnit + 12;

		let rect = renderer.makeRectangle(x, y, gridUnit, gridUnit);
		rect.fill = color.rgb;
		return rect;
	}

	function drawConnection(rectA: Rectangle, rectB: Rectangle, color: PixelColor) {
		let x1 = rectA.position.x;
		let y1 = rectA.position.y;
		let x2 = rectB.position.x;
		let y2 = rectB.position.y;

		let points = [
			new Two.Anchor(x1, y1, 0, 0, -48, 0, Two.Commands.move),
			new Two.Anchor(x2, y2, 48, 0, 0, 0, Two.Commands.curve)
		];
		let bezierPath = new Two.Path(points);
		bezierPath.noFill();
		bezierPath.dashes = [4, 4];
		bezierPath.linewidth = 2;
		bezierPath.stroke = color.rgb;
		bezierPath.automatic = false;
		renderer.add(bezierPath);
		return bezierPath;
	}

	function initializeRenderer() {
		let params = {
			fullscreen: false
		};
		let two = new Two(params).appendTo(worksurface);
		two.height = worksurface.clientHeight;
		two.width = worksurface.clientWidth;
		return two;
	}
</script>

<div id="worksurface" bind:this={worksurface} on:mousemove={onMouseMove}>
	<div class="side-bar">
		<Panel>
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
			<Color color="#ffffffff" size="24px" />
		</Panel>
	</div>
</div>

<style lang="scss" scoped>
	#worksurface {
		position: relative;
		background-size: 24px 24px;
		background-image: linear-gradient(to right, $dark-grey 1px, transparent 1px),
			linear-gradient(to bottom, $dark-grey 1px, $darker-grey 1px);
		height: 100%;

		.side-bar {
			position: absolute;
			display: flex;
			margin: 2px;
		}
	}
</style>
