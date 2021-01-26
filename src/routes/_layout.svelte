<script>
	import Header from '../components/UI/Header.svelte';

	export let segment;
</script>

<Header />

<main>
	<slot></slot>
</main>