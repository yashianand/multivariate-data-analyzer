
<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	let instances;
	let wineQualities = {};
	let features = ["Alchohol", "Total Sulphur Dioxide", "Density", "Volatile Acidity", "PH", "Citric Acid", "Fixed Acidity", "Residual Sugar", "Chlorides", "Free Sulfur Dioxide", "Sulphates", "Quality"]
	let minMax;
	const numClasses = 6;

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
		// set the dimensions and margins of the graph
		const margin = {top: 30, right: 50, bottom: 10, left: 50},
		width = 460 - margin.left - margin.right,
		height = 400 - margin.top - margin.bottom;

		// append the svg object to the body of the page
		const svg = d3.select("#score-distributions-view")
		.append("svg")
		.attr("width", width + margin.left + margin.right)
		.attr("height", height + margin.top + margin.bottom)
		.append("g")
		.attr("transform",
				`translate(${margin.left},${margin.top})`);

		// Parse the Data
		d3.csv("https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/iris.csv").then( function(data) {

			// Color scale: give me a specie name, I return a color
			const color = d3.scaleOrdinal()
				.domain(["Quality 8", "versicolor", "virginica" ])
				.range([ "#440154ff", "#21908dff", "#fde725ff"])

			// Here I set the list of dimension manually to control the order of axis:
			var dimensions = ["Petal_Length", "Petal_Width", "Sepal_Length", "Sepal_Width"]
			var domains = [[0,8], [0,10], [0,20], [0,30]]
			// For each dimension, I build a linear scale. I store all in a y object
			const y = {}
			for (const i in dimensions) {
				var name = dimensions[i]
				y[name] = d3.scaleLinear()
				.domain( domains[i] ) // --> Same axis range for each group
				// --> different axis range for each group --> .domain( [d3.extent(data, function(d) { return +d[name]; })] )
				// .domain( [d3.extent(data, function(d) { return +d[name]; })] )
				.range([height, 0])
			}

			// Build the X scale -> it find the best position for each Y axis
			var x = d3.scalePoint()
				.range([0, width])
				.domain(dimensions);

			// Highlight the specie that is hovered
			const highlight = function(event, d){
				var selected_specie = d.Species

				// first every group turns grey
				d3.selectAll(".line")
				.transition().duration(200)
				.style("stroke", "lightgrey")
				.style("opacity", "0.2")
				// Second the hovered specie takes its color
				d3.selectAll("." + selected_specie)
				.transition().duration(200)
				.style("stroke", color(selected_specie))
				.style("opacity", "1")
			}

			// Unhighlight
			const doNotHighlight = function(event, d){
				console.log('UnHighlight')
				d3.selectAll(".line")
				.transition().duration(200).delay(1000)
				.style("stroke", function(d){ return( color(d.Species))} )
				.style("opacity", "1")
			}

			// The path function take a row of the csv as input, and return x and y coordinates of the line to draw for this raw.
			function path(d) {
				return d3.line()(dimensions.map(function(p) { return [x(p), y[p](d[p])]; }));
			}

			// Draw the lines
			svg
				.selectAll("myPath")
				.data(data)
				.join("path")
				.attr("class", function (d) { return "line " + d.Species } ) // 2 class for each line: 'line' and the group name
				.attr("d",  path)
				.style("fill", "none" )
				.style("stroke", function(d){ return( color(d.Species))} )
				.style("opacity", 0.5)
				.on("mouseover", highlight)
				.on("mouseleave", doNotHighlight )

			// Draw the axis:
			svg.selectAll("myAxis")
				// For each dimension of the dataset I add a 'g' element:
				.data(dimensions).enter()
				.append("g")
				.attr("class", "axis")
				// I translate this element to its right position on the x axis
				.attr("transform", function(d) { return `translate(${x(d)})`})
				// And I build the axis with the call function
				.each(function(d) { d3.select(this).call(d3.axisLeft().ticks(5).scale(y[d])); })
				// Add axis title
				.append("text")
				.style("text-anchor", "middle")
				.attr("y", -9)
				.text(function(d) { return d; })
				.style("fill", "black")

		})
	}

	onMount(async () => {
		const fetched = await fetch("static/Wines.json");
		instances = (await fetched.json()).data;

		const fetched2 = await fetch("static/minmax.json");
		minMax = (await fetched2.json())

		console.log(minMax)
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
	});

	

</script>

<main>
	<h1>Group 2</h1>

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
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Parallel Coordinates</div>
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

</style>