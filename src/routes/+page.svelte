<script lang="ts">
	import Azimuth from './Azimuth.svelte';
</script>

<svelte:head>
	<title>azimuth</title>
	<meta name="description" content="azimuth test" />

	<meta name="viewport" content="width=device-width,initial-scale=1">
	<!-- <meta name="viewport" content="width=320,initial-scale=1"> -->

</svelte:head>

<div class="app">
<h1>方位角</h1>
		<Azimuth />
</div>

<style>
h1 {
	font-size: 1rem;
}
</style>