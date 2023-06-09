<script lang="ts">
	import CommentInput from "$components/CommentInput.svelte";
	import { flip } from "svelte/animate";
	import { handleAddCommentToast } from "$lib/utils/handleToast";
	import type { CommentFromServer } from "$lib/types.js";
	import { parseMarkdown } from "$utils/parseMarkdown";
	import Comment from "$components/Comment.svelte";
	import * as config from "$lib/config";
	import ImageModal from "$components/modals/ImageModal.svelte";

	export let data;
	export let form;

	let pendingLocalComments: CommentFromServer[] = [];

	$: ({ post, comments } = data);

	$: form?.status && handleAddCommentToast(form.status, form.body);

	$: if (comments.data) {
		comments.data.sort((a: CommentFromServer, b: CommentFromServer) => {
			return b.id - a.id;
		});
	}

	let commentCount = 20;

	function handleOptimisticUpdate(event: CustomEvent<CommentFromServer>) {
		pendingLocalComments.push(event.detail);
	}
</script>

<svelte:head>
	<title>{config.title} | {post?.attributes.title}</title>
</svelte:head>

{#if post === null}
	<p>Sorry, this post could not be found.</p>
{:else}
	<article class="post">
		<ImageModal {post} />
		<section class="post-body">
			{@html parseMarkdown(post.attributes.body)}
		</section>
	</article>
	<section class="comment-section">
		<CommentInput
			on:commentAddedOptimistically={(e) => handleOptimisticUpdate(e)}
			formResponseStatus={form?.status}
		/>
		{#each [...pendingLocalComments, ...comments.data] as comment (comment.id)}
			<div animate:flip={{ duration: 900 }}>
				<Comment {comment} />
			</div>
		{/each}
	</section>
{/if}

<style>
	:global(.post-body) {
		margin: 0 auto;
		max-width: 66ch;
		text-align: justify;
		font-size: var(--font-size-M, 1rem);
	}

	:global(.post-body :is(h1, h2, h3, h4, h5, h6)) {
		font-size: var(--font-size-3XL, 1.875rem);
		margin-block-start: 3rem;
		text-align: left;
		line-height: 1.2;
	}

	.comment-section {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		max-width: 48rem;
		margin: 0 auto;
	}
</style>
