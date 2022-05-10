<script context="module">
	const valid_lists = new Set(['news', 'newest', 'show', 'ask', 'jobs']);

	/** @type {import('@sveltejs/kit').Load} */
	export async function load({ params, fetch }) {
		const list = params.list === 'top' ? 'news' : params.list === 'new' ? 'newest' : params.list;

		if (!valid_lists.has(list)) {
			console.log(`invalid list parameter ${list}`);
			return {
				status: 404,
				error: 'Not found'
			};
		}

		const page = +params.page;

		const res = await fetch(`https://api.hnpwa.com/v0/${list}/${page}.json`);
		const items = await res.json();

		return {
			props: {
				page,
				list,
				items
			}
		};
	}
</script>

<script>
	import ItemSummary from './_ItemSummary.svelte';

	export let page;
	export let list;
	export let items;

	const PAGE_SIZE = 30;
	const PAGE_MAX = 10;

	$: start = 1 + (page - 1) * PAGE_SIZE;
	$: isFirstPage = page === 1;
	$: isLastPage = page === PAGE_MAX;
	$: next = isLastPage ? '' : `/${list}/${+page + 1}`;
	$: last = isFirstPage ? '' : `/${list}/${+page - 1}`;

	function goLast() {
		if(page >= 2) {
			window.location.href = last;
		}
	}

	function goNext() {
		if(page <= 9) {
			window.location.href = next;
		}
	}

</script>

<svelte:head>
	<title>Svelte Hacker News</title>
	<meta name="description" content="Latest Hacker News stories in the {list} category" />
</svelte:head>

{#each items as item, i}
	{#if item}
		<!-- sometimes we get bad data? TODO investigate -->
		<ItemSummary {item} index={start + i} />
	{/if}
{/each}

{#if next}
	<div class="pagi">
		<a class="pagi-button" href={last} class:disabled={isFirstPage}>{'<'}</a>
		<span><b>{page}</b> / {PAGE_MAX}</span>
		<a class="pagi-button" href={next} class:disabled={isLastPage}>{'>'}</a>
	</div>
{/if}

<style>
	.pagi {
		width: 8em;
		margin: 3em auto;
		display: flex;
		justify-content: space-between;
		align-items: center;
		color: #ff6600;
	}
	
	.pagi-button.disabled {
		color: #ccc;
		cursor: not-allowed;
	}
	
	.pagi-button {
		display: block;
		cursor: pointer;
		text-decoration: none;
	}
</style>
