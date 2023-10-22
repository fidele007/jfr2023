<script lang="ts">
	import SessionCard from '$lib/SessionCard.svelte';
	import { normalizeString } from '$lib/Constants.svelte';
	import { onMount } from 'svelte';
	import { base } from '$app/paths';

	const logoUrl = `${base}/jfr_2023.png`;

	let loading = true;
	let selectedDate: string = '';
	let eventJson: any;
	let sessionsByDate: any = {};
	let filteredSessions: any;
	let timeout: ReturnType<typeof setTimeout>;

	const debounce = (callback: Function, wait = 300) => {
		return (...args: any[]) => {
			clearTimeout(timeout);
			timeout = setTimeout(() => callback(...args), wait);
		};
	};

	const searchSessions = async (value: string) => {
		// console.log('Searching for: ', value);
		filteredSessions = null;
		loading = true;

		// await new Promise(r => setTimeout(r, 3000));

		const scopedSessions = sessionsByDate[selectedDate];
		filteredSessions = value
			? scopedSessions.filter((x: any) =>
					normalizeString(x.title).toUpperCase().includes(normalizeString(value).toUpperCase())
			  )
			: scopedSessions;

		loading = false;
	};

	const onSearch = async (event: Event) => {
		const value = (event.target as HTMLInputElement).value;
		loading = true;
		debounce(searchSessions).call(null, value);
	};

	const onDateChange = async (date: string) => {
		selectedDate = date;
		await searchSessions((document.getElementById('search') as HTMLInputElement).value);
	};

	onMount(async () => {
		const response = await fetch(`${base}/json/event.json`);
		eventJson = await response.json();

		sessionsByDate[''] = eventJson.data.event.search.items;
		sessionsByDate['2023-10-12'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-12')
		);
		sessionsByDate['2023-10-13'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-13')
		);
		sessionsByDate['2023-10-14'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-14')
		);
		sessionsByDate['2023-10-15'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-15')
		);
		sessionsByDate['2023-10-16'] = eventJson.data.event.search.items.filter((x: any) =>
			x.start.startsWith('2023-10-16')
		);

		await searchSessions((document.getElementById('search') as HTMLInputElement).value);
	});
</script>

<svelte:head>
	<title>JFR 2023</title>
</svelte:head>

<main>
	<div>
		<img src={logoUrl} class="logo" alt="JFR Logo" />
		<h1>JFR 2023</h1>
	</div>
	<div id="search-container">
		<input id="search" type="text" placeholder="Rechercher" on:input={onSearch} />
	</div>

	<ul id="menu">
		<li>
			<a
				href="/"
				class={selectedDate ? '' : 'selected'}
				on:click|preventDefault={() => onDateChange('')}>Tout</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-12' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-12')}>jeu. 12 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-13' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-13')}>ven. 13 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-14' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-14')}>sam. 14 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-15' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-15')}>dim. 15 oct.</a
			>
		</li>
		|
		<li>
			<a
				href="/"
				class={selectedDate == '2023-10-16' ? 'selected' : ''}
				on:click|preventDefault={() => onDateChange('2023-10-16')}>lun. 16 oct.</a
			>
		</li>
	</ul>

	{#if loading}
		<div id="loader">
			<div class="ls-particles ls-part-1" />
			<div class="ls-particles ls-part-2" />
			<div class="ls-particles ls-part-3" />
			<div class="ls-particles ls-part-4" />
			<div class="ls-particles ls-part-5" />
			<div class="lightsaber ls-left ls-green" />
			<div class="lightsaber ls-right ls-red" />
		</div>
	{/if}

	<div class="card-container">
		{#if !loading}
			{#if filteredSessions && filteredSessions.length > 0}
				{#each filteredSessions as item}
					<SessionCard info={item} />
				{/each}
			{:else}
				<p>Aucun résultat</p>
			{/if}
		{/if}
	</div>
</main>

<style>
	main {
		max-width: 80%;
		min-width: 80%;
		margin: 0 auto;
		padding: 2rem;
		text-align: center;
	}

	.logo {
		height: 6em;
		will-change: filter;
		transition: filter 300ms;
	}
	.logo:hover {
		filter: drop-shadow(0 0 2em #646cffaa);
	}

	#search-container {
		margin-bottom: 1em;
		border-radius: 5px;
	}

	#search-container input {
		height: 24px;
		padding: 6px 12px;
		line-height: 1.5;
		font-size: 14px;
		min-width: 300px;
	}

	ul#menu li {
		display: inline;
	}

	a.selected {
		text-decoration: underline;
	}

	.card-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	@media (max-width: 1000px) {
		main {
			max-width: 100%;
		}
	}
</style>
