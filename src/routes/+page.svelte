<script lang="ts">
	import { cubicIn, cubicOut } from "svelte/easing";
	import { slide } from "svelte/transition";

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
	{#if results.length > 0}
		<div
			id="results-screen"
			in:slide={{ duration: 700, easing: cubicOut }}
			out:slide={{ duration: 700, easing: cubicIn }}
		>
			<h1>Results:</h1>
			<button
				id="copy-button"
				onclick={() =>
					navigator.clipboard.writeText(results.join(", "))}
				>Copy</button
			>
			<div id="results-container">
				{#each results as result}
					<p class="result">{result}</p>
				{/each}
			</div>
			<button onclick={() => (results.length = 0)}> Return </button>
		</div>
	{:else}
		<div
			id="generator-screen"
			in:slide={{ duration: 700, easing: cubicOut }}
			out:slide={{ duration: 700, easing: cubicIn }}
		>
			<p id="statement">
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
					<option value="without replacement"
						>without replacement</option
					>
				</select>
				<br />

				between
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
				<br />

				<select bind:value={incl}>
					<option value="including">including</option>
					<option value="not including">not including</option>
				</select>
			</p>
			<button
				onclick={() => generate()}
				{disabled}
			>
				Generate
			</button>
		</div>
	{/if}
</div>

<style>
	@font-face {
		font-family: "Spicy Rice";
		src: url(/Spicy_Rice/SpicyRice-Regular.ttf) format("truetype");
		font-weight: 400;
		font-style: normal;
	}

	:global(body) {
		--bg: #f5f5dc;
		--text: #7b3f00;
		--primary: #b7410e;
		--secondary: #da9100;
		--tertiary: #568203;
		--light-text: #f5f5dc;
		--shadow: 0 4px 12px 0 rgba(0, 0, 0, 0.08),
			0 6px 20px 0 rgba(0, 0, 0, 0.06);

		font-family: "Spicy Rice", serif;
		background-color: var(--bg);
		color: var(--text);
	}
	#page {
		height: 100dvh;
		width: 100dvw;

		font-size: 3em;

		display: flex;
		flex-direction: column;
		align-items: center;
	}

	#statement {
		padding-top: 1em;
		text-align: center;
		margin-top: 0;
	}

	input,
	select {
		padding: 0;
		margin: 0;
		background-color: transparent;

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
	input {
		text-align: center;
	}
	select {
		text-align: left;
		text-align-last: center;
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
		position: absolute;
		bottom: -1em;

		appearance: none;
		border: none;

		background-color: transparent;
		color: var(--secondary);
	}
	button:disabled {
		opacity: 60%;
	}
	button:hover {
		color: var(--tertiary);
	}
	button:hover:disabled {
		color: var(--secondary);
	}
	#generator-screen {
		position: relative;
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	#results-screen {
		width: 100%;
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	#results-container {
		width: 100%;
		height: 50dvh;
		justify-self: center;
		overflow-x: auto;

		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(1em, 2em));
		gap: 2rem;

		place-content: center;
		justify-items: center;
	}

	h1 {
		margin-bottom: -0.1em;
		color: var(--primary);
	}
	.result {
		margin: 0;
		padding: 0;
	}
	#copy-button {
		position: relative;
		top: 0.5em;
		left: -13em;
		font-size: 0.5em;
		transform: rotate(-45deg);
	}
	@media (max-width: 640px) {
		#page {
			font-size: 2.6em;
		}
		#statement {
			margin-top: 1.5em;
		}
		input,
		select {
			line-height: 1.5em;

			text-underline-offset: 8px;
			text-decoration-thickness: 5px;
		}
		#left,
		#right {
			width: 30%;
		}
		#results-container {
			height: 45dvh;
		}
		h1 {
			margin-bottom: 0.7em;
		}
	}
</style>
