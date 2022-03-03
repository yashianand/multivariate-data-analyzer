<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	let instances;

	function setupRadarChart(){
		var marksCanvas = document.getElementById("marksChart");

		var marksData = {
			labels: ["Alchohol", "Total Sulphur Dioxide", "Density", "Chemistry", "Biology", "History"],
			datasets: [{
				label: "Student A",
				backgroundColor: "rgba(200,0,0,0.2)",
				data: [65, 75, 70, 80, 60, 80]
			}, {
				label: "Student B",
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
		console.log(instances)
		setupRadarChart()
	});

	

</script>

<main>
	<h1>Visual Analytics HW 3</h1>

	<div id="container">
		<div id="upper-section">
			<!-- <div id="main-section"> -->
				<div id="Comparison of The Features" class="view-panel" style="width: 1100px; height: 430px">
					<div class="view-title">Comparison of The Features over the Wine Qualities</div>
					<svg >


					</svg>
				</div>
			<!-- </div> -->
			<div id="filter-view" class="view-panel" style="width: 300px;">
				<div class="view-title">Filter View</div>
				<div id="selected-image-view-content">
	
	
				</div>
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<canvas id="marksChart" width="600" height="400"></canvas>
			</div>
		</div>
		<div id="lower-section">
			<!-- <div id="sidebar" > -->
				<div id="efficiency-enhancer" class="view-panel" style="width: 700px; height:250px">
					<div class="view-title">Efficiency Enhancer Scores</div>
					<svg >
						

					</svg>
				</div>
			<!-- </div> -->
			<div id="selected-image-view" class="view-panel" style="width: 700px; height:250px">
				<div class="view-title">Radar Chart</div>
				<div id="selected-image-view-content">


				</div>
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
		flex-direction: column;
	}
	#upper-section, #lower-section {
		display: flex;
		flex-direction: row;
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