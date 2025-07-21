<script>
	import { themes } from '../config.js';
	import { getContext } from 'svelte';

	export let theme = getContext('theme');
</script>

<section style="color: {themes[theme].text}; background-color: '#034c36'">
	<div class="col-medium">
		<slot></slot>
	</div>
</section>