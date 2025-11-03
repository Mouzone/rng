<script lang="ts">
	let numbersToGenerate = $state(1);
	let including = $state("including");
	let minimum = $state(0);
	let maximum = $state(0);
	const results: Number[] = $state([]);

	function generate() {
		results.length = 0;
		for (let i = 0; i < numbersToGenerate; i += 1) {
			const result =
				Math.floor(Math.random() * (maximum - minimum + 1)) + minimum;
			results.push(result);
		}
	}
</script>

<p>
	Generate
	<input
		type="number"
		bind:value={numbersToGenerate}
	/>
	numbers between and
	<select bind:value={including}
		><option>including</option>
		<option>not including</option></select
	><input
		type="number"
		bind:value={minimum}
	/>
	and
	<input
		type="number"
		bind:value={maximum}
	/>
</p>
<div>
	{#each results as result}
		<p>{result}</p>
	{/each}
</div>

<button onclick={generate}> Generate </button>
