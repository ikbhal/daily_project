<script>
import {onMount} from 'svelte';
//https://svelte.dev/tutorial/writable-stores
import {writable} from 'svelte/store';
// import EnvTest from './EnvTest.svelte';

import {db} from './firebase';
let todayDateString = new Date().toISOString().slice(0,10);
export let bathDate= todayDateString;
export let takenBath=false;
export let requireBath=true;// does he not to take bath compulsory(paaak or not)

export let bathList=writable([]);

// https://vim.fandom.com/wiki/Cut/copy_and_paste_using_visual_selection -> V visual select line, d delete, y copy, p paste
async function loadBathList(){
	let bathsRef = db.collection('twd-bath-track');
  	let allBaths= await bathsRef.get();
	let bathListCopy = [];
	for(const doc of allBaths.docs){
		let docData = doc.data();
		let bathCopy = {
			id:doc.id, 
			bathDate:docData.bathDate, 
			requiredBath:docData.requireBath, 
			takenBath:docData.takenBath
		};
		bathListCopy.push(bathCopy);
   		console.log(doc.id, '=>', doc.data());
  	}
	bathList.set(bathListCopy);
}

onMount(async () => {
	loadBathList();
});

function handleOnSubmit() {
	console.log("form submitted");
	db.collection("twd-bath-track").doc(bathDate).set({
	    bathDate,	
		takenBath,
		requireBath
	})
	.then(() => {
		console.log("Document successfully written!");
		loadBathList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});
}

function deleteBath(id){
	console.log("remove bath id:", id);
	db.collection("twd-bath-track").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadBathList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>
	<!-- <EnvTest></EnvTest> -->
<h1>Towheed Bath Accountability</h1>
<p>1. add bath, 2.list bath</p>

<form on:submit|preventDefault={handleOnSubmit}>
	
	<h3>Add Bath</h3>
	<label>Bath Date:<input type="date" bind:value={bathDate}/></label>
	<label><input type=checkbox bind:checked={takenBath}/>Have Taken Bath</label>
	<label><input type=checkbox bind:checked={requireBath}/>Require Bath</label>
	<button type="submit">Save</button>
</form>

<hr/>
<h3>List Bath</h3>
<div class="bathList">
	{#each $bathList as bath}
		<div>
			{bath.bathDate} <br/>
			Taken Bath : {bath.takenBath?'Yes': 'No'}  
			Required bath: {bath.requireBath?'Yes':'No'}
			<button on:click={deleteBath(bath.id)}>Delete</button>
		</div>
	{/each}
</div>
</main>

<style>

</style>