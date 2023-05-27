<script lang="ts">
	import { flip } from "svelte/animate";
	import type { Posts } from "$lib/types";
	import CarouselPost from "./CarouselPost.svelte";
	import { crossfade } from "svelte/transition";
	import { quintOut } from "svelte/easing";
	import Icon from "@iconify/svelte";

	export let posts: Posts = [];
	export let maxPosts = 4;

	const [send, receive] = crossfade({
		duration: (d) => Math.sqrt(d * 200),

		fallback(node, { delay = 0 }) {
			const style = getComputedStyle(node);
			const transform = style.transform === "none" ? "" : style.transform;

			return {
				duration: 1000,
				easing: quintOut,
				delay: delay * 150, // staggered delay based on index
				css: (t) => `
        opacity: ${t};
        transform: ${transform} scale(${t});
      `
			};
		}
	});

	let startIndex = 0;

	// create timeout function on next and prev to ensure that the animation is complete before changing the index

	let isTransitioning = false;

	function next() {
		if (isTransitioning) return;
		isTransitioning = true;
		if (startIndex + maxPosts < posts.length) {
			startIndex += 4;
		} else {
			startIndex = 0;
		}
		setTimeout(() => {
			isTransitioning = false;
		}, 900);
	}

	function prev() {
		if (isTransitioning) return;
		isTransitioning = true;

		if (startIndex > 0) {
			startIndex -= 4;
		} else {
			startIndex = Math.max(posts.length - maxPosts, 0);
		}
		setTimeout(() => {
			isTransitioning = false;
		}, 900);
	}

	$: visiblePosts = posts.slice(startIndex, startIndex + maxPosts);
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->

<div class="container">
	<button on:click|stopPropagation={prev} class="button buttons-desktop">
		<span class="visually-hidden">Prev</span>
		<Icon icon="material-symbols:arrow-circle-left-outline-rounded" width="50" />
	</button>
	<div class="posts">
		{#each visiblePosts as post, i (post.id)}
			<div
				in:receive|local={{ key: post.id, delay: i }}
				out:send|local={{ key: post.id }}
				animate:flip={{ duration: 1000 }}
			>
				<CarouselPost {post} />
			</div>
		{/each}
	</div>
	<button on:click|stopPropagation={next} class="button buttons-desktop next">
		<span class="visually-hidden">Next</span>
		<Icon icon="material-symbols:arrow-circle-left-outline-rounded" width="50" hFlip />
	</button>
	<div class="buttons-mobile">
		<button on:click|stopPropagation={prev} class="button">
			<span class="visually-hidden">Prev</span>
			<Icon icon="material-symbols:arrow-circle-left-outline-rounded" width="50" />
		</button>
		<button on:click|stopPropagation={next} class="button next">
			<span class="visually-hidden">next</span>
			<Icon icon="material-symbols:arrow-circle-left-outline-rounded" width="50" hFlip />
		</button>
	</div>
</div>

<style>
	.container {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		margin-block-end: 1rem;
	}

	.button {
		background-color: transparent;
		border: none;
		outline: none;
		color: var(--color-accent);
		left: -1rem;
		top: 50%;
		cursor: pointer;
		transition: all 0.2s ease-in-out;
	}

	.buttons-desktop {
		display: none;
	}

	.buttons-mobile {
		display: flex;
		justify-content: space-around;
		margin-top: 1rem;
		width: 100%;
	}

	.button.next {
		left: unset;
		right: 0rem;
	}

	.button:hover {
		transform: scale(1.1);
	}

	.posts {
		position: relative;
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 0.5rem;
		width: 100%;
	}

	@media (min-width: 48rem) {
		.buttons-desktop {
			display: flex;
		}

		.buttons-desktop:hover {
			filter: brightness(1.2);
		}
		.buttons-mobile {
			display: none;
		}
		.posts {
			padding: 1rem;
			grid-template-columns: repeat(4, 1fr);
			gap: 1rem;
		}
		.container {
			flex-direction: row;
		}
	}
</style>