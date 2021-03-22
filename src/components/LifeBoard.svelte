<!-- This is needed to change the grid array. -->
<svelte:options accessors/>
<script>
	import { onMount } from 'svelte';

    let canvas;

    export let rows = 16;
    export let columns = 16;
	export let grid = gridForSize(columns, rows);
	export let gridBytes = [];
	export let stateString = null;

	function gridForSize(columns, rows) {
		return Array(Number(columns)).fill(false).map(element => Array(Number(rows)).fill(false));
	}

	function drawGrid(canvas, grid, columns, rows) {
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

		drawDots(canvas, columns, rows);

		ctx.translate(-0.5, -0.5);
	}

	function drawDot(canvas, col, row, columns, rows) {
		let ctx = canvas.getContext
("2d");
		ctx.beginPath();
		let boxWidth = canvas.clientWidth / columns;
		let boxHeight = canvas.clientHeight / rows;
		let centerX = boxWidth * (col + 0.5);
		let centerY = boxHeight * (row + 0.5);
		let radius = Math.min(boxWidth, boxHeight) / 2 - 2;

		ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI, false);
		ctx.fill();
		ctx.stroke();
	}

	function drawDots(canvas, columns, rows) {
		for (var col = 0; col < columns; col++) {
			for (var row = 0; row <
 rows; row++) {
				if (grid[col][row]) {
					drawDot(canvas, col, row, columns, rows);
				}

			}
		}
	}

	export function updateContent() {
		drawGrid(canvas, grid, columns, rows);
		gridBytes = getContentBytes();
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
	}8

	function getContentBytes() {
		let bytes = new Uint8ClampedArray(Math.ceil((columns * rows) / 8.0));
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
		return bytes;
	}

	function setContentBytes(bytes) {
		for (let row = 0; row < rows; row++) {
			for (let col = 0; col < columns; col++) {
				let boardPos = (row * columns) + col;
				let byteIndex = Math.floor(boardPos / 8);
				let bitIndex = boardPos % 8;

				if (byteIndex < bytes.length) {
					let bitValue = bytes[byteIndex] & 1 << (7 - bitIndex);
					grid[col][row] = bitValue > 0;
				} else {
					grid[col][row] = false;
				}
			}
		}
		updateContent();
	}

	function getStateString() {
		let bytes = gridBytes;
		let c = columns;
		let r = rows;

		let hexString = Array.from(bytes, byte => byte.toString(16).padStart(2, '0')).join('');
		let boardString = `life:${c}x${r},${hexString}`
		return boardString;
	}

	function parseStateString(string) {
		const regex = /life:(\d+)x(\d+),([0-9a-fA-F]*)$/g;

		let matches = regex.exec(string);
		if (matches && matches.length == 4) {
			let newCols = matches[1];
			let newRows = matches[2];
			let hexBytes = matches[3];
			let newBytes = new Uint8ClampedArray(Math.ceil(hexBytes.length / 2));
			for (var i = 0; i < newBytes.length; i++) {
				newBytes[i] = parseInt(hexBytes.substr(i * 2, 2), 16);
			}

			if (newCols > 0 && newRows > 0 && newBytes.length > 0) {
				if (columns != newCols || rows != newRows) {
					columns = newCols;
					rows = newRows;
					grid = gridForSize(newCols, newRows);
				}
				setContentBytes(newBytes);
			} else {
				console.log(`Didn't parse valid board data from string ${string}`);
			}
		}
	}

	// Trigger when state string is set
	$: {
		parseStateString(stateString);
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