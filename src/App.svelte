<script>
import {onMount} from 'svelte';
import {writable} from 'svelte/store';
import {db} from './firebase';

let todayDateString = new Date().toISOString().slice(0,10);
let date= todayDateString;

class Project {
	constructor(id, date, name, description, users){
		this.id = id;
		this.date = date;
		this.name = name;
		this.description = description;
		this.users = users;
	}
}
let pcount = 0;
let p1 = new Project(pcount, date, "", "");

let projectStore = writable([]);


onMount(async () => {
	loadProjectList();
});

function createEmptyProject(){
	return   new Project(pcount, date, "", "");
}

async function loadProjectList(){
	console.log("oad project list");
	let collRef = db.collection('daily_projects');
  	let allDocs= await collRef.get();
	let listCopy = [];
	console.log("allDocs:", allDocs);
	for(const doc of allDocs.docs){
		console.log("doc ikb", doc);
		let docData = doc.data();
		let docCopy = new Project(
			doc.id, 
			docData.date, 
			docData.name,
			docData.description
		);
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
		<div> 
			n:{p1.name} d:{p1.description}
		</div>
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