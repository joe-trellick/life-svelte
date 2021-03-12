<script>
	import { onMount } from 'svelte';

    let canvas;

    export let rows = 16;
    export let columns = 16;
	export const grid = Array(Number(columns)).fill(false).map(element => Array(Number(rows)).fill(false));
	export let gridBytes = [];
	export let stateString = null;

	grid[5][9] = true;
	grid[0][11] = true;

	function drawGrid(canvas, grid) {
		let ctx = canvas.getContext("2d");

		// Clear existing content
		ctx.clearRect(0, 0, canvas.clientWidth, canvas.clientHeight);

		ctx.translate(0.5, 0.5);  // get rid of line-center blurriness
		ctx.lineWidth = 0.5;  // hairlines on retina?
		ctx.strokeStyle = "#999999";

		for (var row = 1; row < rows; row++) {
			let y = row * canvas.clientHeight / rows; 
			for (var col = 1; col < columns; col++) {
				let x = col * canvas.clientHeight / columns;
				ctx.beginPath();
				ctx.moveTo(x, 0);
				ctx.lineTo(x, canvas.clientHeight);
				ctx.stroke();
			}
			ctx.beginPath();
			ctx.moveTo(-0.5, y);
			ctx.lineTo(canvas.clientWidth, y);
			ctx.stroke();
		}

		drawDots(canvas);

		ctx.translate(-0.5, -0.5);
	}

	function drawDot(canvas, x, y) {
		let ctx = canvas.getContext("2d");
		ctx.beginPath();
		let boxWidth = canvas.clientWidth / columns;
		let boxHeight = canvas.clientHeight / rows;
		let centerX = boxWidth * (x + 0.5);
		let centerY = boxHeight * (y + 0.5);
		let radius = Math.min(boxWidth, boxHeight) / 2 - 2;

		ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI, false);
		ctx.fill();
		ctx.stroke();
	}

	function drawDots(canvas) {
		for (var col = 0; col < columns; col++) {
			for (var row = 0; row < rows; row++) {
				if (grid[col][row]) {
					drawDot(canvas, col, row);
				}
			}
		}
	}

	export function updateContent() {
		drawGrid(canvas, grid, rows, columns);
		getContentBytes();
		stateString = getStateString();
	}

	onMount(() => {
		canvas.width = canvas.clientWidth;
		canvas.height = canvas.clientHeight;

		updateContent();
	});

	function boardPositionForScreenPosition(x, y) {
		const col = Math.floor(x / (canvas.clientWidth / columns));
		const row = Math.floor(y / (canvas.clientHeight / rows));
		return {col: col, row: row};
	}

	function click(event) {
		const boardPos = boardPositionForScreenPosition(event.offsetX, event.offsetY);
	
		grid[boardPos.col][boardPos.row] = !grid[boardPos.col][boardPos.row];
		updateContent();
	}

	function getContentBytes() {
		let bytes = new Uint8ClampedArray(Math.ceil((rows * columns) / 8.0));
		for (let row = 0; row < rows; row++) {
			for (let col = 0; col < columns; col++) {
				let boardPos = (row * columns) + col;
				let byteIndex = Math.floor(boardPos / 8);
				let bitIndex = boardPos % 8;

				if (grid[col][row]) {
					bytes[byteIndex] |= 1 << 7 - bitIndex;
				}
			}
		}
		console.log(bytes);
		gridBytes = bytes;
	}

	function getStateString() {
		let bytes = gridBytes;
		let c = columns;
		let r = rows;

		let hexString = bytes.map((byte) => {
			return byte.toString(16).padStart(2, '0');
		}).join('');
		let boardString = `life:${c}x${r},${hexString}`
		console.log(`boardString = ${boardString}`);
		return boardString;
	}

</script>

<canvas bind:this={canvas} on:click={click}/>

<style>
    canvas {
		width: 640px;
		height: 640px;
		border: 1px solid #000000;
	}
</style>