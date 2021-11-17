<script>
import {onMount} from 'svelte';
import {writable} from 'svelte/store';
import {db} from './firebase';

let todayDateString = new Date().toISOString().slice(0,10);
let date= todayDateString;

class Project {
	constructor(id, name, details, users){
		this.id = id;
		this.name = name;
		this.details = name;
		this.users = users;
	}
}
let pcount = 0;
let p1 = new Project(pcount, "untitle project", "write description here");

let projectStore = writable([]);


onMount(async () => {
	loadProjectList();
});

function createEmptyProject(){
	return   new Project(pcount, "untitle project", "write description here");
}

async function loadProjectList(){
	let collRef = db.collection('daily-projects');
  	let allDocs= await CollRef.get();
	let listCopy = [];
	for(const doc of allDocs.docs){
		let docData = doc.data();
		let docCopy = {
			id:doc.id, 
			date:docData.date, 
			name:docData.name,
			description: docData.description
		};
		listCopy.push(docCopy);
   		console.log(doc.id, '=>', doc.data());
  	}
	projectStore.set(listCopy);
}

function handleOnSubmit() {
	console.log("handle on ssubmit");
	db.collection("daily_projects").add({
		date,
		name: p1.name,
		description: p1.description

	})
	.then(() => {
		console.log("Document successfully written!");
		loadProjectList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});


	$projectStore = [... $projectStore, p1];
	p1 = createEmptyProject(); 
}


function deleteDoc(id){
	console.log("remove id:", id);
	db.collection("daily_projects").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadProjectList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>
	<h1>daily project</h1>
	<p>used for free teacher </p>
	<p>add , delete, edit, lists, search daily project, assign project</p>

	<h3>Projects</h3>
	<h4>Add Project</h4>
	<form on:submit|preventDefault={handleOnSubmit}>
		<label for="pname">Name:</label>
		<input type="text" name="pname" bind:value={p1.name}/>
		<br/>
		<label for="pdesc">Description</label>
		<input type="text" name="pdesc" bind:value={p1.description}  />
		<br/>
		<button type="submit">Add</button>
	</form>
	<h4>Project List</h4>
	<div class="projectList">
		{#each $projectStore as p}
		<div class="project">
			<div class="project_name">
			{p.name}
			</div>
			<div class="project_details">
			{p.description}
			</div>
			<div class="actions">
				<button on:click={() => console.log("delete")}>Delete</button>
				<button on:click={() => deleteDoc(p.id)}>Edit</button>
			</div>
		</div>
		{/each}
	</div>
</main>

<style>
</style>