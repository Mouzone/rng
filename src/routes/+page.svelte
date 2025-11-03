<script lang="ts">
	let numbersToGen = $state(1);
	let incl = $state("including");
	let withRep = $state("with replacement");
	let min = $state(0);
	let max = $state(0);
	const results: Number[] = $state([]);

	function generate() {
		results.length = 0;
		for (let i = 0; i < numbersToGen; i += 1) {
			const offset = incl === "including" ? 1 : 0;
			const effectiveMin = min + offset;
			const effectiveMax = max - offset;
			const result =
				Math.floor(Math.random() * (effectiveMax - effectiveMin)) +
				effectiveMin;
			results.push(result);
		}
	}
</script>

<p>
	Generate
	<input
		type="number"
		bind:value={numbersToGen}
	/>
	number(s)
	<select bind:value={withRep}>
		<option>with replacement</option>
		<option>without replacement</option>
	</select>
	between and
	<select bind:value={incl}
		><option>including</option>
		<option>not including</option></select
	><input
		type="number"
		bind:value={min}
	/>
	and
	<input
		type="number"
		bind:value={max}
	/>
</p>
<div>
	{#each results as result}
		<p>{result}</p>
	{/each}
</div>

<button onclick={generate}> Generate </button>
