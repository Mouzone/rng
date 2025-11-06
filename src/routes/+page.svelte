<script lang="ts">
	import "normalize.css";

	let dialogElement: HTMLDialogElement | null = $state(null);
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

	let disabled = $derived(
		(() => {
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
		})()
	);

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
		dialogElement?.showModal();
	}
</script>

<div id="page">
	<dialog bind:this={dialogElement}>
		<div id="dialog-contents">
			<div id="results-container">
				{#each results as result}
					<p>{result}</p>
				{/each}
			</div>

			<button onclick={() => dialogElement?.close()}> Return </button>
		</div>
	</dialog>

	<div id="generator">
		<p>
			Generate
			<br />

			<input
				inputmode="numeric"
				bind:value={numbersToGenInput}
				oninput={(e) => (numbersToGenInput = handleInput(e))}
			/>
			<br />
			{#if numbersToGen === 1}
				number
			{:else}
				numbers
			{/if}
			<br />
			<select bind:value={withRep}>
				<option value="with replacement">with replacement</option>
				<option value="without replacement">without replacement</option>
			</select>
			<br />
			between and
			<br />

			<select bind:value={incl}>
				<option value="including">including</option>
				<option value="not including">not including</option>
			</select>
			<br />

			<input
				inputmode="numeric"
				id="left"
				bind:value={minInput}
				oninput={(e) => (minInput = handleInput(e))}
			/>
			and
			<input
				inputmode="numeric"
				id="right"
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
</div>

<style>
	@font-face {
		font-family: "Spicy Rice";
		src: url(/Spicy_Rice/SpicyRice-Regular.ttf) format("truetype");
		font-weight: 400;
		font-style: normal;
	}

	#page {
		--bg: #f5f5dc;
		--text: #7b3f00;
		--primary: #b7410e;
		--secondary: #da9100;
		--tertiary: #568203;
		--light-text: #f5f5dc;
		--shadow: 0 4px 12px 0 rgba(0, 0, 0, 0.08),
			0 6px 20px 0 rgba(0, 0, 0, 0.06);

		height: 100dvh;
		width: 100dvw;

		font-family: "Spicy Rice", serif;
		background-color: var(--bg);
		color: var(--text);

		font-size: 3em;

		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	#generator {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	p {
		text-align: center;
		margin-bottom: 0.5em;
	}

	input,
	select {
		padding: 0;
		margin: 0;
		background-color: transparent;
		text-align: center;
		appearance: none;
		border: none;
		outline: none;
		line-height: 1.5em;

		color: var(--primary);
		text-decoration-line: underline;
		text-decoration-style: wavy;
		text-decoration-color: var(--primary);
		text-underline-offset: 5px;
		text-decoration-thickness: 6px;

		font-family: inherit;
		font-size: inherit;
	}

	#left {
		text-align: right;
		width: 40%;
	}

	#right {
		text-align: left;
		width: 40%;
	}

	button {
		appearance: none;
		border: none;

		background-color: transparent;
		color: var(--primary);
		text-decoration-line: underline;
		text-decoration-style: wavy;
		text-decoration-color: var(--primary);

		text-underline-offset: 5px;
		text-decoration-thickness: 6px;
	}
	button:disabled {
		opacity: 60%;
	}
	button:hover {
		color: var(--secondary);
		text-decoration-color: var(--secondary);
	}
	button:hover:disabled {
		color: var(--primary);
		text-decoration-color: var(--primary);
	}

	dialog {
		padding: 0.5em;
		width: 70%;
		border: none;
		border-radius: 30px;
		box-shadow: var(--shadow);
		background-color: var(--bg);
		color: var(--text);
	}

	dialog::backdrop {
		background-color: rgba(0, 0, 0, 0.4);
		backdrop-filter: blur(2px);
	}

	#dialog-contents {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	#results-container {
		width: 90%;
		display: flex;
		gap: 0.5em;
		overflow-x: auto;
		align-items: center;
		justify-content: center;
	}
</style>
