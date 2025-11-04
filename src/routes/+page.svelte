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

		if (target.value.length === 0) {
			return "0";
		}

		const digitsOnly = target.value.replace(/\D/g, "");
		const noLeadingZeros = digitsOnly.replace(/^0+/, "");

		if (digitsOnly.length > 0 && noLeadingZeros.length === 0) {
			return "0";
		}

		return noLeadingZeros;
	}

	function generate() {
		results.length = 0;

		const actualMin = incl === "including" ? min : min + 1;
		const actualMax = incl === "including" ? max : max - 1;
		const span = actualMax - actualMin + 1;

		while (results.length < numbersToGen) {
			let result = Math.floor(Math.random() * span) + actualMin;

			if (withRep === "without replacement") {
				while (results.includes(result)) {
					result = Math.floor(Math.random() * span) + actualMin;
				}
			}

			results.push(result);
		}
	}
</script>

<div id="page">
	<div id="generator">
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

		<button
			onclick={generate}
			{disabled}
		>
			Generate
		</button>
	</div>

	<div style="display: flex; gap: 1em;">
		{#each results as result}
			<p>{result}</p>
		{/each}
	</div>
</div>

<style>
	#page {
		height: 100dvh;
		width: 100dvw;

		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	#generator {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	p {
		text-align: center;
	}
	input,
	select {
		text-align: center;
		margin: 0, 1em, 1em, 0;
	}
	input {
		width: 2em;
	}
	select {
		width: 12em;
	}
</style>
