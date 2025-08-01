<script context="module">
  // Based on svelte-scroller by Rich Harris
  // https://github.com/sveltejs/svelte-scroller
  // Patched to transpile to IE 11 and allow for split-screen view option

	const handlers = [];
	let manager;

	if (typeof window !== 'undefined') {
		const run_all = () => handlers.forEach(fn => fn());

		window.addEventListener('scroll', run_all);
		window.addEventListener('resize', run_all);
	}

	if (typeof IntersectionObserver !== 'undefined') {
		const map = new Map();

		const observer = new IntersectionObserver((entries, observer) => {
			entries.forEach(entry => {
				const update = map.get(entry.target);
				const index = handlers.indexOf(update);

				if (entry.isIntersecting) {
					if (index === -1) handlers.push(update);
				} else {
					update();
					if (index !== -1) handlers.splice(index, 1);
				}
			});
		}, {
			rootMargin: '400px 0px' // TODO why 400?
		});

		manager = {
			add: ({ outer, update }) => {
				const { top, bottom } = outer.getBoundingClientRect();

				if (top < window.innerHeight && bottom > 0) handlers.push(update);

				map.set(outer, update);
				observer.observe(outer);
			},

			remove: ({ outer, update }) => {
				const index = handlers.indexOf(update);
				if (index !== -1) handlers.splice(index, 1);

				map.delete(outer);
				observer.unobserve(outer);
			}
		};
	} else {
		manager = {
			add: ({ update }) => {
				handlers.push(update);
			},

			remove: ({ update }) => {
				const index = handlers.indexOf(update);
				if (index !== -1) handlers.splice(index, 1);
			}
		};
	}
</script>

<script>
	import { onMount } from 'svelte';

	// config
	export let top = 0;
	export let bottom = 1;
	export let threshold = 0.5;
	export let query = 'section';
	export let parallax = false;

	// bindings
	export let index = 0;
	export let count = 0;
	export let offset = 0;
	export let progress = 0;
	export let visible = false;
	export let splitscreen = false; 
	export let id = null;

	let outer;
  let bgContainer;
	let foreground;
	let background;
	let left;
	let sections;
	let wh = 0;
	let fixed;
	let offset_top;
	let width = 1;
	let height;
	let inverted;

	$: top_px = Math.round(top * wh);
	$: bottom_px = Math.round(bottom * wh);
	$: threshold_px = Math.round(threshold * wh);

	$: (top, bottom, threshold, parallax, update());

	onMount(() => {
		sections = foreground.querySelectorAll(query);
		count = sections.length;

		update();

		const scroller = { outer, update };

		manager.add(scroller);
		return () => manager.remove(scroller);
	});

	
  function setFixed() {
    if (bgContainer) {
      let style = `position: ${fixed ? 'fixed' : 'absolute'}; top: 0; transform: translate(0, ${offset_top}px); width: ${width}px; z-index: ${inverted ? 3 : 1};`;
      bgContainer.style.cssText = style;
    }
  }

	function update() {
		if (!foreground) return;

		
		const bcr = outer.getBoundingClientRect();
		left = bcr.left;
		width = bcr.right - bcr.left;

		
		const fg = foreground.getBoundingClientRect();
		const bg = background.getBoundingClientRect();

		visible = fg.top < wh && fg.bottom > 0;

		const foreground_height = fg.bottom - fg.top;
		const background_height = bg.bottom - bg.top;

		const available_space = bottom_px - top_px;
		progress = (top_px - fg.top) / (foreground_height - available_space);

		if (progress <= 0) {
			offset_top = 0;
      if (fixed) {
        fixed = false;
        setFixed();
      } 
		} else if (progress >= 1) {
			offset_top = parallax
				? (foreground_height - background_height)
				: (foreground_height - available_space);
      if (fixed) {
        fixed = false;
        setFixed();
      }
		} else {
			offset_top = parallax ?
				Math.round(top_px - progress * (background_height - available_space)) :
				top_px;
      if (!fixed) {
        fixed = true;
        setFixed();
      }
		}

		for (index = 0; index < sections.length; index += 1) {
			const section = sections[index];
			const { top } = section.getBoundingClientRect();

			const next = sections[index + 1];
			const bottom = next ? next.getBoundingClientRect().top : fg.bottom;

			offset = (threshold_px - top) / (bottom - top);
			id = section.dataset.id ? section.dataset.id : null;
			if (bottom >= threshold_px) break;
		}
	}
</script>

<svelte:window bind:innerHeight={wh}/>

<svelte-scroller-outer bind:this={outer} class:splitscreen>
	<svelte-scroller-background-container class='background-container' bind:this={bgContainer}>
		<svelte-scroller-background bind:this={background}>
			<slot name="background"></slot>
		</svelte-scroller-background>
	</svelte-scroller-background-container>

	<svelte-scroller-foreground bind:this={foreground}>
		<slot name="foreground"></slot>
	</svelte-scroller-foreground>
</svelte-scroller-outer>

<style>
	svelte-scroller-outer {
		display: block;
		position: relative;
		max-width: 100%;
		color: pink; 
	}

	svelte-scroller-background {
		display: block;
		position: relative;
		width: 100%;
		color: #FAD9F9;
	}

	svelte-scroller-foreground {
		display: block;
		position: relative;
		z-index: 2;
	}

	svelte-scroller-foreground::after {
		content: ' ';
		display: block;
		clear: both;
	}

	svelte-scroller-background-container {
		display: block;
		position: absolute;
		width: 100%;
		max-width: 100%;
		pointer-events: none;
		
		will-change: transform;
		
	}
</style>