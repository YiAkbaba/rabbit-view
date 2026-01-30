<script>
	import { goto } from '$app/navigation';
	import { store, pb } from '$lib/store.svelte.js';

	let { rabbitID = '' } = $props();

	let rabbit = $state({
		name: 'New Name',
		rabbithole: ''
	});

	let rabbitholes = $state([]);

	let wrongRabbitName = $derived(rabbit.name.length > 0 && rabbit.name[0] !== 'J');

	async function addRabbit() {
		await store.addRabbit(rabbit);
		goto('/');
	}

	async function saveChanges() {
		await store.editRabbit(rabbitID, rabbit);
		goto('/');
	}

	$effect(async () => {
		rabbitholes = await pb.collection('rabbitholes').getFullList();
		if (rabbitID) {
			const rec = await pb.collection('rabbits').getOne(rabbitID);
			rabbit = { name: rec.name, rabbithole: rec.rabbithole ?? '' };
		}
	});
</script>

<div class="flex flex-col gap-2">
	{#if rabbitID}
		<h1 class="text-lg font-bold">Edit rabbit {rabbitID}</h1>
	{:else}
		<h1 class="text-lg font-bold">Add a rabbit</h1>
	{/if}

	<label class="input">
		<span class="label">Name</span>
		<input type="text" class="grow" bind:value={rabbit.name} />
	</label>

	<div>
		<label class="select"
			><span class="label">Rabbithole</span>
			<select bind:value={rabbit.rabbithole}>
				{#each rabbitholes as rabbithole (rabbithole.id)}
					<option value={rabbithole.id}>{rabbithole.name}</option>
				{/each}
			</select></label
		>
	</div>

	{#if wrongRabbitName}
		<div role="alert" class="mt-4 alert alert-error">
			<span>Watch out! Rabbit names must start with "J"!</span>
		</div>
	{/if}

	{#if rabbitID}
		<button
			class="btn btn-primary"
			onclick={saveChanges}
			disabled={wrongRabbitName || rabbit.name.length === 0}
		>
			Save Changes!
		</button>
	{:else}
		<button
			class="btn btn-primary"
			onclick={addRabbit}
			disabled={wrongRabbitName || rabbit.name.length === 0}
		>
			Add Rabbit!
		</button>
	{/if}
</div>
