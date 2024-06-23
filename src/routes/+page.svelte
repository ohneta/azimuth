<script lang="ts">
	import Azimuth from './Azimuth.svelte';

	let drawOrthAngle: bool = false;

</script>

<svelte:head>
	<title>方位角</title>
	<meta name="description" content="azimuth" />

	<meta name="viewport" content="width=device-width,initial-scale=1">
	<!-- <meta name="viewport" content="width=320,initial-scale=1"> -->

</svelte:head>

<div class="app">
	<h1>方位角</h1>

	<label>
		<input type="checkbox" bind:checked={drawOrthAngle} />
		直行表示
	</label>

{#if drawOrthAngle}
	<Azimuth drawOrthAngle={drawOrthAngle} />
{:else}
	<Azimuth drawOrthAngle={drawOrthAngle} />
{/if}

</div>

<style>
h1 {
	font-size: 1rem;
}
</style>