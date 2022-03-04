<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { schemeCategory10 } from "d3-scale-chromatic";
	import { tSNE } from "./tsne";

	let instances = [];
	let wineQualities = {};
	let features = ["Alchohol", "Total Sulphur Dioxide", "Density", "Volatile Acidity", "PH", "Citric Acid", "Fixed Acidity", "Residual Sugar", "Chlorides", "Free Sulfur Dioxide", "Sulphates"]
	const numClasses = 6;

	// tSNE variable definitions
	let instance_features = [];
	let projection = [];
	let xScale, yScale;
	let canvasSizeProjection = 300;
	let colorScale;
	// tSNE definitions end here

	function setupRadarChart(){
		var marksCanvas = document.getElementById("marksChart");

		var marksData = {
			labels: features.slice(0, 6),
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
				backgroundColor: "rgba(254,229,217,0.8)",
				data: [0.8231051906, 0.4405991789, 1, 1, 1, 0.4372159091]
			}]
		};

		var radarChart = new Chart(marksCanvas, {
			type: 'radar',
			data: marksData
		});
	}

	function setupParallelCoordinates(){
		//TODO
	}

	function setupTSNE() {
		let opt = {epsilon: 10}; // epsilon is learning rate (10 = default)
		let tsne = new tSNE(opt); // create a tSNE instance

		// initialize data. Here we have 3 points and some example pairwise dissimilarities
		for (let i in instances) {
			let value = []
			for (let key in instances[i]) {
				if (key !== "Id" && key!== "quality") {
					value.push(instances[i][key])
				}
			}
			instance_features.push(value)
		}
		tsne.initDataRaw(instance_features);

		for(let k = 0; k < 500; k++) {
		tsne.step(); // every time you call this, solution gets better
		}
		projection = tsne.getSolution(); // projection is an array of 2-D points that you can plot
	}

	onMount(async () => {
		const fetched = await fetch("static/Wines.json");
		instances = (await fetched.json()).data;
		for (let k = 3; k < numClasses+3; ++k) {
			wineQualities[k] = {
				'instances': []
			}
		}
		instances.forEach(instance => {
			wineQualities[instance.quality]['instances'].push(instance)
		});
		setupRadarChart()
		setupParallelCoordinates()
		console.log('All instances: ', instances)
		console.log('Wine Qualities: ', wineQualities)

		setupTSNE()
		console.log('tSNE Projections: ', projection)
		xScale = scaleLinear()
			.domain([
				0,
				Math.max(...projection.map(record => record[0]))
			])
			.range([0, canvasSizeProjection]);
		yScale = scaleLinear()
			.domain([
				Math.min(...projection.map(record => record[1])),
				Math.max(...projection.map(record => record[1]))])
			.range([0, canvasSizeProjection]);

			colorScale = scaleOrdinal(schemeCategory10)
			.domain(instances.map(row => row["quality"]));

	});



</script>

<main>
	<h1>Multivariate Data Analyzer</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="radar-view" class="view-panel">
				<div class="view-title">Radar Chart</div>
				<canvas id="marksChart" width="600" height="400"></canvas>
			</div>
			<div id="input-view" class="view-panel">
				<div class="view-title">Input View</div>
				<div id="input-view-content">
					<svg height="340">
						{#each features as label,i}
							<text x="10" y="{i*30+15}" width="80%" height="10">{label}</text>
							<foreignObject x="170" y="{i*30}" width="170" height="20">
								<input type="text"/>
							</foreignObject>
						{/each}
						<rect x="10" y="310" width="300" height="30" fill="red"></rect>
						<text x="90" y="330" width="300" height="30" fill="white">Predict Quality</text>
					</svg>
				</div>
			</div>
		</div>

		<div id="main-section" style="width: 1000px;">
			<div id="parallel-coordinates-view" class="view-panel">
				<div class="view-title">Parallel Coordinates</div>
				<svg >


				</svg>
			</div>
			<div id="tSNE-projection-view" class="view-panel">
				<div class="view-title">tSNE Projection View</div>
				<svg class="view-tsne" x="1000" y="100">
					{#each instances as record}
						<circle
							style="fill: {colorScale(record["quality"])};"
							cx={xScale(projection[record['Id']][0])+350}
							cy={yScale(projection[record['Id']][1])+50}
							r='3'
							opacity= '0.4'
						/>
					{/each}
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
		height: 70%;
		width: 98%;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 4px 4px 4px;
	}
	#input-view-content {
		height: 370px;
	}
	.view-tsne {
		width: 800px;
		height: 400px;
	}



</style>
