
<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	let instances;
	let wineQualities = {};
	const numClasses = 6;

	function setupRadarChart(){
		var marksCanvas = document.getElementById("marksChart");

		var marksData = {
			labels: ["Alchohol", "Total Sulphur Dioxide", "Density", "Volatile Acidity", "PH", "Citric Acid"],
			datasets: [{
				label: "Quality 8",
				backgroundColor: "rgba(165,15,21,0.5)",
				data: [1, 0.5917909543, 0.9977424972, 0.4786131524, 0.9615133085, 1]
			}, {
				label: "Quality 7",
				backgroundColor: "rgba(222,45,38,0.5)",
				data: [0.9480313834, 0.6196717882, 0.9986368143, 0.4566643279, 0.9684384252, 0.9592565098]
			}, {
				label: "Quality 6",
				backgroundColor: "rgba(251,106,74,0.5)",
				data: [0.8788761964,0.7231826107,0.9991489043,0.5624469486,0.9764779577,0.700119336]
			}, {
				label: "Quality 5",
				backgroundColor: "rgba(252,146,114,0.5)",
				data: [0.8185333654, 1, 0.9996387108, 0.6523924432, 0.9726158343, 0.6230601722]
			}, {
				label: "Quality 4",
				backgroundColor: "rgba(252,187,161,0.5)",
				data: [0.848744594, 0.6413510818, 0.9990761098, 0.7845814179, 0.9951469788, 0.4452722985]
			}, {
				label: "Quality 3",
				backgroundColor: "rgba(254,229,217,0.5)",
				data: [0.8231051906, 0.4405991789, 1, 1, 1, 0.4372159091]
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
				'instances': []
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
			<div id="radar-view" class="view-panel">
				<div class="view-title">Radar Chart</div>
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