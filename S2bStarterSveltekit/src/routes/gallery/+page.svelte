
<script>
	// `data` holds what load() in +page.server.js returned.
	let { data } = $props();

	// $state makes these reactive: the page updates when they change.
	let name = $state('');
	let species = $state('');
	let light = $state('Indirect light');
	let submitting = $state(false);
	let submitError = $state('');
	let listError = $state('');

	// Our own copy of the list, seeded from load()'s data, so the page can
	// change straight after an add, status change or delete without a reload.
	let plants = $state(data.plants);

	// ADD — the form. A plain fetch() to OUR OWN /api/plants route
	// (src/routes/api/plants/+server.js), never straight to Express.
	// SvelteKit forwards it to Express.
	async function handleSubmit(event) {
		event.preventDefault(); // stop the browser reloading the page
		submitting = true;
		submitError = '';

		try {
			// await means: pause here until the server answers, then carry on.
			const res = await fetch('/api/plants', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ name, species, light, swapped: false })
			});

			if (!res.ok) {
				throw new Error(`Request failed with status ${res.status}`);
			}

			// The API sends back the saved plant (with its id). Add it to our list.
			const newPlant = await res.json();
			plants.push(newPlant);

			name = '';
			species = '';
			light = 'Indirect light';
		} catch (err) {
			submitError = 'Could not add that plant. Is the API running?';
			console.error(err);
		} finally {
			submitting = false;
		}
	}

	// UPDATE — the swapped status. A PATCH to OUR OWN /api/plants/<id>
	// route (src/routes/api/plants/[id]/+server.js), sending only the
	// field that changes.
	async function toggleSwapped(plant) {
		listError = '';

		try {
			const res = await fetch(`/api/plants/${plant.id}`, {
				method: 'PATCH',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ swapped: !plant.swapped })
			});

			if (!res.ok) {
				throw new Error(`Request failed with status ${res.status}`);
			}

			// The API sends back the updated plant. Copy the new value onto
			// the plant in our list and the status changes.
			const updated = await res.json();
			plant.swapped = updated.swapped;
		} catch (err) {
			listError = 'Could not update that plant. Is the API running?';
			console.error(err);
		}
	}

	// DELETE — the ✕ button. A successful delete answers 204 (No Content),
	// so there is no body to read.
	async function deletePlant(plant) {
		listError = '';

		try {
			const res = await fetch(`/api/plants/${plant.id}`, {
				method: 'DELETE'
			});

			if (!res.ok) {
				throw new Error(`Request failed with status ${res.status}`);
			}

			// Rebuild our list without that plant: filter keeps every plant
			// whose id is NOT the one we deleted.
			plants = plants.filter((p) => p.id !== plant.id);
		} catch (err) {
			listError = 'Could not delete that plant. Is the API running?';
			console.error(err);
		}
	}
</script>

<main>
	<h1>Plant Swap Shelf</h1>
	<p><a href="/">← Home</a></p>

	{#if listError}
		<p class="error">{listError}</p>
	{/if}

	<ul class="plant-list">
		{#each plants as plant (plant.id)}
			<li class="plant-card">
				<div>
					<strong>{plant.name}</strong>
					<span class="species">{plant.species}</span>
					<span class="light">Light: {plant.light}</span>
				</div>
				<div class="buttons">
					<button
						class="icon-button"
						class:is-swapped={plant.swapped}
						onclick={() => toggleSwapped(plant)}
						title="Toggle swapped status"
					>
						{plant.swapped ? 'Swapped' : 'Available'}
					</button>
					<button
						class="icon-button delete"
						onclick={() => deletePlant(plant)}
						title="Delete this plant"
					>
						✕
					</button>
				</div>
			</li>
		{/each}
	</ul>

	<h2>Add a plant</h2>
	<form onsubmit={handleSubmit}>
		<label>
			Name
			<input type="text" bind:value={name} required />
		</label>
		<label>
			Species
			<input type="text" bind:value={species} required />
		</label>
		<label>
			Light requirement
			<select bind:value={light}>
				<option>Low light</option>
				<option>Indirect light</option>
				<option>Bright indirect light</option>
				<option>Direct sunlight</option>
			</select>
		</label>
		<button type="submit" disabled={submitting}>
			{submitting ? 'Adding…' : 'Add plant'}
		</button>
		{#if submitError}
			<p class="error">{submitError}</p>
		{/if}
	</form>
</main>


<style>
	main {
		max-width: 36rem;
		margin: 2rem auto;
		padding: 0 1.5rem;
		font-family: system-ui, sans-serif;
	}
	.sketch-list {
		list-style: none;
		padding: 0;
	}
	.sketch-card {
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-left: 6px solid #ccc;
		background: #f8f8f8;
		padding: 0.75rem 1rem;
		margin-bottom: 0.5rem;
		border-radius: 4px;
	}
	.artist {
		color: #666;
		margin-left: 0.5rem;
		font-size: 0.9rem;
	}
	.buttons {
		display: flex;
		gap: 0.25rem;
	}
	.icon-button {
		background: none;
		border: none;
		font-size: 1.5rem;
		cursor: pointer;
		color: #ccc;
	}
	.icon-button.is-favourite {
		color: #f5a623;
	}
	.icon-button.delete:hover {
		color: #b91c1c;
	}
	form {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		max-width: 20rem;
	}
	label {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		font-size: 0.9rem;
	}
	.error {
		color: #b91c1c;
	}
</style>
