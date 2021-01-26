<script context="module">
    import meetups from "../meetups-store";

	export function preload({params, query}){
		return this.fetch('https://svelte-course-45e6f-default-rtdb.firebaseio.com/meetups.json')
			.then(res => {
				if(!res.ok){
					throw new Error('Fallo hijo de tptm');
				}
				return res.json();
			})
			.then(data => {
				const loadedMeetups = [];
				for(const key in data){
					loadedMeetups.push({
						...data[key],
						id: key
					});
				}
				return {loadedMeetups: loadedMeetups.reverse()};
			})
			.catch(err => {
				this.error(500, "Perra barata");
				console.log(err)
			});
	}
</script>

<script>
    import { flip } from "svelte/animate";
    import { scale } from "svelte/transition";
    import EditMeetup   from "../components/Meetups/EditMeetup.svelte";
    import MeetupGrid   from "../components/Meetups/MeetupGrid.svelte";
    import MeetupItem   from "../components/Meetups/MeetupItem.svelte";
    import Button       from "../components/UI/Button.svelte";
    import MeetupFilter from "../components/Meetups/MeetupFilter.svelte";
    import LoadingSpinner from "../components/UI/LoadingSpinner.svelte";
    import Error from "../components/UI/Error.svelte";
	import { onDestroy, onMount } from "svelte";

    
    let editMode = null;
    let page = "overview";
    let pageData = {};
    let favOnlys = false;
    let isLoading = false;
	let error = null;
	export let loadedMeetups;
	let meetupsStore = [];
	let unsubscribe = null;

    $: filteredMeetups = favOnlys ? meetupsStore.filter(m => m.isFavorite) : meetupsStore;

    onMount(() => {
		unsubscribe = meetups.subscribe(items => meetupsStore = items);
		meetups.setMeetups(loadedMeetups);
	});

	onDestroy(() => unsubscribe ? unsubscribe() : null);

    function handleSubmit(event){
        editMode = null;
    }

    function cancelEdit(){
        editMode = null;
        pageData = {};
    }

    function showDetails(event){
        page = "details";
        pageData.id = event.detail;
    }

    function closeDetails(){
        page = "overview";
        pageData = {};
    }

    function handleEdit(event){
        editMode = "edit";
        pageData.id = event.detail;
    }

    function setFilter(event){
        favOnlys = event.detail === 1;
    }

    function handleCancelError(){
        error = null;
    }
</script>

<style>
    main{
        margin-top: 5rem;
    }

    .meetup-controls{
        margin: 1rem;
    }
    .no-meetups{
        margin: 1rem;
    }
</style>

{#if error}
    <Error message={error.message} on:cancel={handleCancelError} />
{/if}

<svelte:head>
	<title>Meetups</title>
</svelte:head>

<main>
	<div class="meetup-controls">
		<Button on:click={() => editMode="edit"} >New Meetup</Button>
		<div>&nbsp;</div>
		<MeetupFilter on:select={setFilter} />
	</div>
	{#if editMode === "edit"}
		<EditMeetup id={pageData.id} on:save={handleSubmit} on:cancel={cancelEdit}/>
	{/if}
	{#if isLoading}
		<LoadingSpinner />
	{:else}
		{#if filteredMeetups.length === 0}
			<p class="no-meetups">No meetups found, start adding someone</p>
		{/if}
		<MeetupGrid>
			{#each filteredMeetups as meetup (meetup.id)}
				<div animate:flip={{duration:300}} transition:scale>
					<MeetupItem 
						id={meetup.id}
						title={meetup.title}
						subtitle={meetup.subtitle}
						description={meetup.description}
						imageUrl={meetup.imageUrl}
						address={meetup.address}
						contactEmail={meetup.contactEmail}
						isFavorite={meetup.isFavorite}
						on:showdetails={showDetails}
						on:edit={handleEdit}
					/>
				</div>
			{/each}
		</MeetupGrid>
	{/if}
</main>