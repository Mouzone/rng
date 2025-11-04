<script lang="ts">
	let numbersToGenInput = $state("1");
	let incl = $state("including");
	let withRep = $state("without replacement");
	let minInput = $state("0");
	let maxInput = $state("0");

	let numbersToGen = $derived(Number(numbersToGenInput) || 1);
	let min = $derived(Number(minInput) || 0);
	let max = $derived(Number(maxInput) || 0);

	let effMin = $derived(incl === "including" ? min : min + 1);
	let effMax = $derived(incl === "including" ? max : max - 1);

	let rangeSize = $derived(Math.max(0, effMax - effMin + 1));

	let disabled = $derived.by(() => {
		if (effMax < effMin) {
			return true;
		}

		if (numbersToGen <= 0) {
			return true;
		}

		if (withRep === "without replacement" && rangeSize < numbersToGen) {
			return true;
		}

		return false;
	});

	let results: number[] = $state([]);

	function handleInput(e: Event) {
		const target = e.currentTarget as HTMLInputElement;
		return target.value.replace(/\D/g, "");
	}

	function generate() {
		results.length = 0;

		// Pre-calculate effective min/max for this generation
		const actualMin = incl === "including" ? min : min + 1;
		const actualMax = incl === "including" ? max : max - 1;
		const span = actualMax - actualMin + 1;

		while (results.length < numbersToGen) {
			let result = Math.floor(Math.random() * span) + actualMin;

			// --- FIX: Logic inverted to "without replacement" ---
			if (withRep === "without replacement") {
				// Keep generating a new number until we find one
				// not already in the results
				while (results.includes(result)) {
					result = Math.floor(Math.random() * span) + actualMin;
				}
			}

			results.push(result);
		}
	}
</script>

<p>
	Generate
	<input
		inputmode="numeric"
		bind:value={numbersToGenInput}
		oninput={(e) => (numbersToGenInput = handleInput(e))}
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
		inputmode="numeric"
		bind:value={minInput}
		oninput={(e) => (minInput = handleInput(e))}
	/>
	and
	<input
		inputmode="numeric"
		bind:value={maxInput}
		oninput={(e) => (maxInput = handleInput(e))}
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
