<script lang="ts">
	let numbersToGen = $state(1);
	let incl = $state("including");
	let withRep = $state("without replacement");
	let min = $state(0);
	let max = $state(0);
	let disabled = $derived.by(() => {
		if (incl === "including") {
			if (withRep === "without replacement") {
				return max - min < 1;
			} else {
				return max - min < numbersToGen;
			}
		} else {
			if (withRep === "without replacement") {
				return max - min < 2;
			} else {
				return max - min - 1 < numbersToGen;
			}
		}
	});
	const results: Number[] = $state([]);

	function handleMinInput(e) {
		const roundedValue = Math.round(e.currentTarget.valueAsNumber);
		const absoluteValue = Math.abs(roundedValue);
		min = absoluteValue;
	}

	function handleMaxInput(e) {
		const roundedValue = Math.round(e.currentTarget.valueAsNumber);
		const absoluteValue = Math.abs(roundedValue);
		max = absoluteValue;
	}
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
		oninput={handleMinInput}
	/>
	and
	<input
		type="number"
		bind:value={max}
		oninput={handleMaxInput}
	/>
</p>
<div style="display: flex; gap: 1em;">
	{#each results as result}
		<p>{result}</p>
	{/each}
</div>

<button
	onclick={generate}
	{disabled}
>
	Generate
</button>
