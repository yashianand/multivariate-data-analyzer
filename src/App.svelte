
<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	let instances;
	let wineQualities = {};
	let averageFeatures = {}
	const numClasses = 6;

	function setupRadarChart(){
		var marksCanvas = document.getElementById("marksChart");

		var marksData = {
			labels: ["Alchohol", "Total Sulphur Dioxide", "Density", "Volatile Acidity", "PH", "Citric Acid"],
			datasets: [{
				label: "Quality 8",
				backgroundColor: "rgba(200,0,0,0.2)",
				data: [65, 75, 70, 80, 60, 80]
			}, {
				label: "Quality 7",
				backgroundColor: "rgba(0,0,200,0.2)",
				data: [54, 65, 60, 70, 70, 75]
			}]
		};

		var radarChart = new Chart(marksCanvas, {
			type: 'radar',
			data: marksData
		});
	}

	onMount(async () => {
		const fetched = await fetch("static/Wines.json");
		instances = (await fetched.json()).data;
		setupRadarChart()
		for (let k = 3; k < numClasses+3; ++k) {
			wineQualities[k] = {
				'instances': [],
				'Average Alchohol': 0,
				'Average Total Sulphur Dioxide': 0,
				'Average Density': 0,
				'Average Volatile Acidity': 0,
				'Average PH': 0,
				'Average Citric Acid': 0
			}
		}
		instances.forEach(instance => {
			wineQualities[instance.quality]['instances'].push(instance)
		});
		console.log('All instances: ', instances)
		console.log('Wine Qualities: ', wineQualities)
	});

	

</script>

<main>
	<h1>Visual Analytics HW 3</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<canvas id="marksChart" width="600" height="400"></canvas>
			</div>
			<div id="selected-image-view" class="view-panel">
				<div class="view-title">Selected Image</div>
				<div id="selected-image-view-content">


				</div>
			</div>
		</div>

		<div id="main-section" style="width: 1000px;">
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Score Distributions</div>
				<svg >


				</svg>
			</div>
		</div>
	</div>
</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
	}
	#selected-image-view-content {
		height: 100px;
		padding: 15px;
	}
	svg {
		margin: 5px;
	}



</style>