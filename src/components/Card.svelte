<script>
	import { fade } from "svelte/transition";
	export let answer = "";
	export let solution = "";
	let displayCalc = false;

	function toggleDisplay(event) {
		displayCalc = !displayCalc;
	}
</script>

<article class="card">
	<section class="answer">
		<div>{@html answer}</div>
		<button class="display-btn" on:click={toggleDisplay}>
			{#if displayCalc}
				<i class="fas fa-caret-up" />
			{:else}<i class="fas fa-caret-down" />
			{/if}
		</button>
	</section>

	{#if displayCalc}
		<section class="solution" transition:fade={{ duration: 500 }}>
			{@html solution}
			<!-- <slot name="solution" /> -->
		</section>
	{/if}
</article>
