<script>
	import LifeBoard from './components/LifeBoard.svelte'
	import { onMount } from 'svelte';

	let lifeBoard;
	let running = false;
	let timerId = null;
	let boardStateString = "";

	onMount(() => {
		lifeBoard.grid[0][0] = true;
		lifeBoard.grid[0][5] = false;

		// here comes the GLIDER
		lifeBoard.grid[2][3] = true;
		lifeBoard.grid[3][4] = true;
		lifeBoard.grid[1][5] = true;
		lifeBoard.grid[2][5] = true;
		lifeBoard.grid[3][5] = true;
		lifeBoard.updateContent();
	});

	function gridStateAt(grid, col, row) {
		if (row < 0 || row >= grid[0].length || col < 0 || col >= grid.length) {
			return false;
		}

		return grid[col][row];
	}

	function advanceState(grid) {
		let cols = grid.length;
		let rows = grid[0].length;

		let changes = [];

		for (let col = 0; col < cols; col++) {
			for (let row = 0; row < rows; row++) {
				let neighbors = 0;
				neighbors += gridStateAt(grid, col - 1, row - 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col, row - 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col + 1, row - 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col + 1, row) ? 1 : 0;
				neighbors += gridStateAt(grid, col + 1, row + 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col, row + 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col - 1, row + 1) ? 1 : 0;
				neighbors += gridStateAt(grid, col - 1, row) ? 1 : 0;

				const current = grid[col][row];
				if (!current && neighbors == 3) {
					changes.push([col, row, true]);
				} else if (current && !(neighbors == 2 || neighbors == 3)) {
					changes.push([col, row, false]);
				}
			}
		}

		changes.forEach(function(change) {
			grid[change[0]][change[1]] = change[2];
		});
	}

	function step() {
		advanceState(lifeBoard.grid);
		lifeBoard.updateContent();
	}

	function startStop() {
		running = !running;

		if (running) {
			timerId = setInterval(step, 500);
		} else {
			clearInterval(timerId);
			timerId = null;
		}
	}
</script>

<main>
	<LifeBoard rows=16 columns=16 bind:this={lifeBoard} bind:stateString={boardStateString} />
	<br/>
{#if !running}
	<button on:click={step}>Step</button>
	<button on:click={startStop}>Run</button>
{:else}
	<button on:click={startStop}>Stop</button>
{/if}

	<br/><br/>
	State: <pre>{boardStateString}</pre>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>