<script lang="ts">
	let numbersToGen = $state(1);
	let incl = $state("including");
	let withRep = $state("without replacement");
	let min = $state(0);
	let max = $derived(min + (incl === "including" ? 1 : 2));
	const results: Number[] = $state([]);

	function generate() {
		results.length = 0;
		while (results.length < numbersToGen) {
			let result;
			if (incl === "including") {
				result = Math.floor(Math.random() * (max - min + 1)) + min;
			} else {
				result = Math.floor(Math.random() * (max - min - 1)) + min + 1;
			}
			if (withRep === "with replacement") {
				while (results.includes(result)) {
					if (incl === "including") {
						result =
							Math.floor(Math.random() * (max - min + 1)) + min;
					} else {
						result =
							Math.floor(Math.random() * (max - min - 1)) +
							min +
							1;
					}
				}
			}

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
		<option value="with replacement">with replacement</option>
		<option value="without replacement">without replacement</option>
	</select>
	between and
	<select bind:value={incl}>
		<option value="including">including</option>
		<option value="not including">not including</option>
	</select>
	<input
		type="number"
		bind:value={min}
	/>
	and
	<input
		type="number"
		bind:value={max}
	/>
</p>
<div style="display: flex; gap: 1em;">
	{#each results as result}
		<p>{result}</p>
	{/each}
</div>

<button onclick={generate}> Generate </button>
