<script lang="ts">
	import { Button, Content, DataTable, Dropdown, TextInput } from 'carbon-components-svelte';
	import Renew from 'carbon-icons-svelte/lib/Renew.svelte';

	let name: string;

	let participants: string[] = [];

    // secret santa
	let allocatedPartners: Map<string, string>;

    // Has the secret santa map been generated?
	let generated = false;

    // Holds name of partner of the selected user in dropdown
	let selectedUsersPartner: string | undefined = '';

    // Checks whether the dummy dropdown has been selected
	let dropDownInitialSelected = false;
    /* Holds the index that was selected for the dummy drop. Becuase we've added an extra dummy variable,
     when we switch to a dropdown without it, we must make sure the selected index is set to the dummy index - 1 */
	let dropDownInitialIndexChange = 0;

	function shuffle<T>(array: T[]) {
		let currentIndex = array.length,
			randomIndex;

		// While there remain elements to shuffle.
		while (currentIndex != 0) {
			// Pick a remaining element.
			randomIndex = Math.floor(Math.random() * currentIndex);
			currentIndex--;

			// And swap it with the current element.
			[array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
		}

		return array;
	}

    // itterates over array and checks if any values match that index of the other
	function matchingIndexes<T>(arr1: T[], arr2: T[]): boolean {
		for (let i = 0; i < arr1.length; i++) {
			if (arr1[i] == arr2[i]) return true;
		}
		return false;
	}

    // Builds a map given two arrays
	function allocatePartners(secretSantas: string[], allocated: string[]): Map<string, string> {
		let santas = new Map<string, string>();
		for (let i = 0; i < secretSantas.length; i++) {
			santas.set(secretSantas[i], allocated[i]);
		}
		return santas;
	}

    // Generate map for secret santa given a list of participants
	function generate(participants: string[]) {
		let pCopy = [...participants];
		do {
			pCopy = shuffle(pCopy);
		} while (matchingIndexes(pCopy, participants));

		allocatedPartners = allocatePartners(participants, pCopy);
	}
</script>

<div>
	<surface-div>
		<h1>Secret Santa Generator</h1>
		<add-person>
			<TextInput
				style={`text-align: center`}
				placeholder="Enter Participant Name..."
				bind:value={name}
				on:keyup={(e) => {
					if (e.key == 'Enter') {
						if (name == '') return;
						participants = [...participants, name];
						name = '';
					}
				}}
			/>
		</add-person>

		<h3>Participants</h3>
		<participant-list>
			{#each participants as participant}
				<participant-name>
					{participant}
				</participant-name>
			{/each}
		</participant-list>
		<Button
			style={`margin-top: 1.5rem;`}
			icon={Renew}
			on:click={() => {
				if (participants.length <= 1) return;
				generate(participants);
				generated = true;
			}}>Generate</Button
		>

		{#if generated}
			<h3>Select Your Name:</h3>
            <!-- Two dropdowns are in place using a conditonal so we can have a default value that dissapears after the first selection -->
			{#if dropDownInitialSelected}
				<Dropdown
					style={`min-width: 30%; margin-top: 1rem;`}
					selectedId={dropDownInitialIndexChange - 1}
					items={[
						...participants.map((participant, index) => {
							return { id: index, text: participant };
						})
					]}
					on:select={(e) => {
						let participant = e.detail.selectedItem.text;

						selectedUsersPartner = allocatedPartners.get(participant);
					}}
				/>
			{:else}
				<Dropdown
					style={`min-width: 30%; margin-top: 1rem;`}
					selectedId={0}
					items={[
						{ id: 0, text: 'Select Your Name' },
						...participants.map((participant, index) => {
							return { id: index + 1, text: participant };
						})
					]}
					on:select={(e) => {
						dropDownInitialSelected = true;
						let participant = e.detail.selectedItem.text;
						dropDownInitialIndexChange = parseInt(e.detail.selectedItem.id);
						console.log(dropDownInitialIndexChange);
						selectedUsersPartner = allocatedPartners.get(participant);
					}}
				/>
			{/if}
			{#if selectedUsersPartner}
				<p>You are buying for: {selectedUsersPartner}</p>
			{/if}
		{/if}
	</surface-div>
</div>

<style>
	:global(html, body, body > *) {
		min-width: 100%;
		width: 100%;
		max-width: 100%;

		min-height: 100%;
		height: 100%;
		max-height: 100%;
	}

	:global(html) {
		padding-bottom: 1rem;
	}

	h1 {
		margin-bottom: 2rem;
		margin-top: 1rem;
	}

	div {
		display: flex;
		justify-content: center;
		align-items: center;
		min-height: 100%;
		flex: 1;
	}
	surface-div {
		background: #ffffff;
		border-radius: 5px;
		width: 40%;
		height: 40%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 1rem;
	}

	add-person {
		width: 40%;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	h3 {
		text-align: center;
		margin-top: 2rem;
	}

	participant-list {
		margin-top: 1rem;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		max-height: 25rem;
		width: 55%;
		overflow: auto;
		padding: 1rem;
	}

	p {
		margin-top: 1rem;
	}
</style>
