<script>
	let { selected = 'pattern1', onSelect } = $props();
</script>

<nav class="sidebar-left">
	<h2>Patterns</h2>
	<ul>
		<li class={selected === 'pattern1' ? 'active' : ''}>
			<a class="patternButton" onclick={() => onSelect('pattern1')}>Pattern 1</a>
		</li>
		<li class={selected === 'pattern2' ? 'active' : ''}>
			<a class="patternButton" onclick={() => onSelect('pattern2')}>Pattern 2</a>
		</li>
	</ul>
</nav>
