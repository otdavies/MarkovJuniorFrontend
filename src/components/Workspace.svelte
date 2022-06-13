<script lang="ts">
	import { onMount } from 'svelte';
	import Two from 'two.js';
	import { ZUI } from 'two.js/extras/jsm/zui';
	import { Group } from 'two.js/src/group';
	import type { Path } from 'two.js/src/path';
	import type { Rectangle } from 'two.js/src/shapes/rectangle';
	import Color from './Color.svelte';
	import Panel from './Panel.svelte';

	let worksurface: HTMLElement;
	let renderer: Two;
	let surface: Group = new Group();
	let gridUnit: number = 24;
	let shape: Rectangle;

	let visualGridSize = 24;
	let visualGridOffset = '24px 24px';

	let pixels: Array<Rectangle> = [];
	let connections: Array<Path> = [];

	onMount(async () => {
		renderer = initializeRenderer();
		let a = drawPixel(5, 5, new PixelColor(255, 0, 0));
		let b = drawPixel(3, 7, new PixelColor(255, 0, 0));
		let c = drawPixel(2, 7, new PixelColor(255, 255, 255));
		let d = drawPixel(6, 5, new PixelColor(255, 255, 255));
		shape = b;
		drawConnection(a, b, new PixelColor(255, 0, 0));
		renderer.add(surface);
		addZUI();
		renderer.update();
	});

	class PixelColor {
		rgb: string;
		constructor(r: number, g: number, b: number) {
			this.rgb = 'rgb(' + r + ', ' + g + ', ' + b + ')';
		}
	}

	function addZUI() {
		let domElement = renderer.renderer.domElement;
		let zui = new ZUI(surface);
		let mouse = new Two.Vector();
		let touches = {};
		let distance = 0;
		let dragging = false;

		zui.addLimits(0.5, 1);

		domElement.addEventListener('mousedown', mousedown, false);
		domElement.addEventListener('mousewheel', mousewheel, false);
		domElement.addEventListener('wheel', mousewheel, false);

		// domElement.addEventListener('touchstart', touchstart, false);
		// domElement.addEventListener('touchmove', touchmove, false);
		// domElement.addEventListener('touchend', touchend, false);
		// domElement.addEventListener('touchcancel', touchend, false);

		function mousedown(e) {
			let rect = shape.getBoundingClientRect();
			mouse.x = Math.floor(Math.abs(e.clientX) / gridUnit) * gridUnit;
			mouse.y = Math.floor(Math.abs(e.clientY) / gridUnit) * gridUnit;
			dragging =
				mouse.x > rect.left && mouse.x < rect.right && mouse.y > rect.top && mouse.y < rect.bottom;
			window.addEventListener('mousemove', mousemove, false);
			window.addEventListener('mouseup', mouseup, false);
		}

		function mousemove(e) {
			let bounds = worksurface.getBoundingClientRect();
			let cx = e.clientX - bounds.left;
			let cy = e.clientY - bounds.top;
			let snapped_cx = Math.floor(Math.abs(cx) / gridUnit) * gridUnit * Math.sign(cx);
			let snapped_cy = Math.floor(Math.abs(cy) / gridUnit) * gridUnit * Math.sign(cy);
			let snapped_dx = snapped_cx - mouse.x;
			let snapped_dy = snapped_cy - mouse.y;

			if (dragging) {
				shape.position.x += snapped_dx / zui.scale;
				shape.position.y += snapped_dy / zui.scale;
			} else {
				zui.translateSurface(snapped_dx, snapped_dy);
			}
			mouse.set(snapped_cx, snapped_cy);
			renderer.update();
		}

		function mouseup(e) {
			window.removeEventListener('mousemove', mousemove, false);
			window.removeEventListener('mouseup', mouseup, false);
		}

		function mousewheel(e) {
			let dy = (e.wheelDeltaY || -e.deltaY) / 1000;
			// zui.zoomBy(dy, e.clientX, e.clientY);
			// visualGridSize = Math.round(zui.scale * 24);
			renderer.update();
		}

		function touchstart(e) {
			switch (e.touches.length) {
				case 2:
					pinchstart(e);
					break;
				case 1:
					panstart(e);
					break;
			}
		}

		function touchmove(e) {
			switch (e.touches.length) {
				case 2:
					pinchmove(e);
					break;
				case 1:
					panmove(e);
					break;
			}
		}

		function touchend(e) {
			touches = {};
			let touch = e.touches[0];
			if (touch) {
				// Pass through for panning after pinching
				mouse.x = touch.clientX;
				mouse.y = touch.clientY;
			}
		}

		function panstart(e) {
			let touch = e.touches[0];
			mouse.x = touch.clientX;
			mouse.y = touch.clientY;
		}

		function panmove(e) {
			let touch = e.touches[0];
			let dx = touch.clientX - mouse.x;
			let dy = touch.clientY - mouse.y;
			zui.translateSurface(dx, dy);
			mouse.set(touch.clientX, touch.clientY);
		}

		function pinchstart(e) {
			for (let i = 0; i < e.touches.length; i++) {
				let touch = e.touches[i];
				touches[touch.identifier] = touch;
			}
			let a = touches[0];
			let b = touches[1];
			let dx = b.clientX - a.clientX;
			let dy = b.clientY - a.clientY;
			distance = Math.sqrt(dx * dx + dy * dy);
			mouse.x = dx / 2 + a.clientX;
			mouse.y = dy / 2 + a.clientY;
		}

		function pinchmove(e) {
			for (let i = 0; i < e.touches.length; i++) {
				let touch = e.touches[i];
				touches[touch.identifier] = touch;
			}
			let a = touches[0];
			let b = touches[1];
			let dx = b.clientX - a.clientX;
			let dy = b.clientY - a.clientY;
			let d = Math.sqrt(dx * dx + dy * dy);
			let delta = d - distance;
			zui.zoomBy(delta / 250, mouse.x, mouse.y);
			distance = d;
		}
	}

	function drawPixel(x: number, y: number, color: PixelColor) {
		x = x * gridUnit + 12;
		y = y * gridUnit + 12;

		let rect = new Two.Rectangle(x, y, gridUnit, gridUnit);
		rect.fill = color.rgb;
		surface.add(rect);
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
		surface.add(bezierPath);
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

<div id="worksurface">
	<div
		class="grid"
		bind:this={worksurface}
		style:background-position={visualGridOffset}
		style:background-size="{visualGridSize}px {visualGridSize}px"
	/>
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
		height: 100%;
		background-color: transparent;
		z-index: 100;

		.grid {
			height: 100%;
			width: 100%;
			background-image: linear-gradient(to right, $dark-grey 1px, transparent 1px),
				linear-gradient(to bottom, $dark-grey 1px, $darker-grey 1px);
		}

		.side-bar {
			position: absolute;
			display: flex;
			margin: 2px;
			top: 0;
		}
	}
</style>
