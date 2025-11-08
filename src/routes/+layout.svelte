<script lang="ts">
	import "normalize.css";
	import favicon from "$lib/assets/favicon.svg";
	import { onMount } from "svelte";

	async function detectSWUpdate() {
		const registration = await navigator.serviceWorker.ready;

		registration.addEventListener("updatefound", () => {
			const newSW = registration.installing;
			newSW?.addEventListener("statechange", () => {
				if (newSW.state === "installed") {
					if (confirm("New update available! Reload to update?")) {
						newSW.postMessage({ type: "SKIP_WAITING" });
						window.location.reload();
					}
				}
			});
		});
	}

	onMount(() => detectSWUpdate());
	let { children } = $props();
</script>

<svelte:head>
	<link
		rel="icon"
		href={favicon}
	/>
</svelte:head>

{@render children()}
