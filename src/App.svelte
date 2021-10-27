<script defer>
	export let name;
	let count = 0;
	let username = window.localStorage.getItem("owBrukernavn");
	let eventDictionary = {};

	async function addIfThere(url, name, type) {
		const res = await fetch(url);
		const json = await res.json();
		json.results.forEach(result => {
			if (result.user.username === username) {
				if(eventDictionary[type] != null) {
					eventDictionary[type] = [...eventDictionary[type], ['https://online.ntnu.no/events/' + result.event, name]]
				} else {
					eventDictionary[type] = [['https://online.ntnu.no/events/' + result.event, name]];
				}
			}
		})
		if (json.next) {
			await addIfThere(json.next);
		}
	}

	async function fetchEvents(url) {
		count += 1;
		const res = await fetch(url);
		const json = await res.json();
		console.log("Fetching page", count+"/"+Math.ceil(json.count/10))
		json.results.forEach(event => {
			console.log(event);
			let type = "";
			console.log(event.event_type);
			switch(event.event_type) {
				case 1:
					type = "Annet";
					break;
				case 2:
					type = "Bedriftspresentasjon";
					break;
				case 3:
					type = "Kurs";
					break;
				default:
					type = "Annet";
					break;
			}
			addIfThere('https://online.ntnu.no/api/v1/event/attendees/?event=' + event.id, event.title, type);
		})
		if (json.next) {
			return await fetchEvents(json.next);
		} else {
			return true;
		}
	}

	function doFetch() {
		if (username) {
			console.log("Starting fetch...")
			fetchEvents(`https://online.ntnu.no/api/v1/events/?ordering=event_start&event_start__gte=${new Date().toISOString().split('T')[0]}`);
		}
	}

	function waitForLoad() {
		let button = document.getElementById('setButton');
		let textInp = document.getElementById('stringInput');
		if (button != null && textInp != null) {
			button.addEventListener('click', setName);
			textInp.addEventListener('change', () => username = textInp.value.toLowerCase())
			if (username) {
				textInp.value = username;
			}
		} else {
			setTimeout(waitForLoad, 100);
		}
	}

	waitForLoad();

	function setName() {
		window.localStorage.setItem("owBrukernavn", username);
		eventDictionary = {};
		doFetch();
	}
	doFetch()

</script>

<main>
	<label for="stringInput">Brukernavn:</label>
	<input id="stringInput" type="text">
	<button id="setButton" >Check</button>
	<h3>Dine kommende arrangementer:</h3>
	<div id="lists">
		{#each Object.entries(eventDictionary) as [type, events]}
			<div id="list">
				<h4>{type}:</h4>
				{#each events as event}
					<a class="owLink" href={event[0]} target="_blank" >{event[1]}</a>
				{/each}
			</div>
		{/each}
	</div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		margin: 0 auto;
		width: 350px;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	#lists {
		display: flex;
		flex-direction: column;
	}
	#list {
		display: flex;
		flex-direction: column;
	}
	.owLink {
		text-decoration: none;
	}
</style>
