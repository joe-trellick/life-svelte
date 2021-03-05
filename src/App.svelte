<script>
	export let name;
	import LifeBoard from './components/LifeBoard.svelte'
	import { onMount } from 'svelte';

	let lifeBoard;

	onMount(() => {
		console.log("LB: " + lifeBoard.grid);
		// Doesn't work without accessors?
		// lifeBoard.rows = 16;
		// lifeBoard.columns = 16;
		lifeBoard.grid[0][0] = true;
		lifeBoard.grid[0][5] = false;
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
		console.log(`grid (${cols} x ${rows})`);

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
				let next = current;
				if (!current && neighbors == 3) {
					// grid[col][row] = true;
					changes.push([col, row, true]);
					next = true;
				} else if (current && !(neighbors == 2 || neighbors == 3)) {
					// grid[col][row] = false;
					changes.push([col, row, false]);
					next = false;
				}
				console.log(`(${col}, ${row}) is ${current} with ${neighbors} neighbors => ${next}`);
			}
		}

		changes.forEach(function(change) {
			grid[change[0]][change[1]] = change[2];
		});
	}

	function step() {
		console.log("Step here");
		advanceState(lifeBoard.grid);
		lifeBoard.updateContent();
	}
</script>

<main>
	<LifeBoard rows=16 columns=16 bind:this={lifeBoard} />
	<br/>
	<button on:click={step}>Step</button>
	<h1>Hello {name}!</h1>
	<p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>