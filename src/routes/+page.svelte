<script>
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';
	import HeaderNavi from '$lib/components/Header_navi.svelte';
	import Pattern from '$lib/components/Pattern4.svelte';
	import Pattern2 from '$lib/components/Pattern5.svelte';

	let selectedPattern = $state('pattern1');
</script>

<div class="app-container">
	<Header />

	<main class="app-main">
		<HeaderNavi selected={selectedPattern} onSelect={(p) => (selectedPattern = p)} />

		{#if selectedPattern === 'pattern1'}
			<Pattern />
		{:else if selectedPattern === 'pattern2'}
			<Pattern2 />
		{/if}
	</main>

	<Footer />
</div>
