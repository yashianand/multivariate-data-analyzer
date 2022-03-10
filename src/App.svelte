<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleSequential, scalePoint } from "d3-scale";
	import { interpolateRdBu } from "d3-scale-chromatic";

	let instances;
	let wineQualities = {};
	let features = ["alcohol", "total sulfur dioxide", "density", "volatile acidity", "pH", "citric acid", "fixed acidity", "residual sugar", "chlorides", "free sulfur dioxide", "sulphates", "quality"]
	let radar_labels = [];
	let minMax;
	let correlation_dict = {};
	let corr_array = [];
	let selectedToggle = 0
	let comparison_values = [];
	let items = [];
	let corrColorScheme;
	let xScale, xScaleTicks, yScale, yScaleTicks;
	let colors = ["#A50F15", "#DE2D26", "#FB6A4A", "#FC9272", "#FCBBA1", "#FEE5D9"]
	let chartSpread, chartSpreadTicks, featureScale;
	const numClasses = 6;
	let key;
	let radar_arr = [];
	let new_radar_arr = [];
	console.log(radar_arr)






	function setupParallelCoordinates(){
		// set the features and margins of the graph
		const margin = {top: 10, right: 5, bottom: 20, left: 30},
		width = 850 - margin.left - margin.right,
		height = 350 - margin.top - margin.bottom;

		// append the svg object to the body of the page
		const svg = d3.select("#parallel")
		.append("svg")
		.attr("width", width + margin.left + margin.right)
		.attr("height", height + margin.top + margin.bottom)
		.append("g")
		.attr("transform",
				`translate(${margin.left},${margin.top})`);

		// Parse the Data
		var data;
		if (selectedToggle == 0) {
			data = instances
		} else {
			data = comparison_values
		}


		// Color scale: give me a specie name, I return a color
		const color = d3.scaleOrdinal()
			.domain(["8", "7", "6" , "5", "4", "3"])
			.range(colors)

		// Here I set the list of dimension manually to control the order of axis:
		// For each dimension, I build a linear scale. I store all in a y object
		const y = {}
		for (const i in features) {
			var name = features[i]
			var mima = minMax[name]
			y[name] = d3.scaleLinear()
			// .domain( [Math.floor(mima.Min), Math.ceil(mima.Max)] )
			.domain( [mima.Min, mima.Max] )
			.range([height, 0])
		}

		// Build the X scale -> it find the best position for each Y axis
		var x = d3.scalePoint()
			.range([0, width])
			.domain(features);

		// Highlight the specie that is hovered
		const highlight = function(event, d){
			var selected_quality = d.quality

			// first every group turns grey
			d3.selectAll(".line")
			.transition().duration(20)
			.style("stroke", "lightgrey")
			.style("opacity", "0.2")
			// Second the hovered specie takes its color
			d3.selectAll(".line" + selected_quality)
			.transition().duration(20)
			.style("stroke", color(selected_quality))
			.style("opacity", "1")
		}

		// Unhighlight
		const doNotHighlight = function(event, d){
			d3.selectAll(".line")
			.transition().duration(50).delay(0)
			.style("stroke", function(d){ return( color(d.quality))} )
			.style("opacity", "1")
		}

		// The path function take a row of the csv as input, and return x and y coordinates of the line to draw for this raw.
		function path(d) {
			return d3.line()(features.map(
				function(p) {
					return [x(p), y[p](d[p])];
				}
			));
		}

		// Draw the lines
		svg
			.selectAll("myPath")
			.data(data)
			.join("path")
			.attr("class", function (d) { return "line " + d.quality } ) // 2 class for each line: 'line' and the group name
			.attr("d",  path)
			.style("fill", "none" )
			.style("stroke", function(d){ return( color(d.quality))} )
			.style("stroke-width", 5)
			.style("opacity", 0.6)
			.on("mouseover", highlight)
			.on("mouseleave", doNotHighlight )

		// Draw the axis:
		svg.selectAll("myAxis")
			// For each dimension of the dataset I add a 'g' element:
			.data(features).enter()
			.append("g")
			.attr("class", "axis")
			// I translate this element to its right position on the x axis
			.attr("transform", function(d) { return `translate(${x(d)})`})
			// And I build the axis with the call function
			.each(function(d) { d3.select(this).call(d3.axisLeft().ticks(5).scale(y[d])); })
			// Add axis title
			.append("text")
			.style("text-anchor", "middle")
			.attr("y", height + 10)
			.text(function(d) { return d; })
			.style("fill", "black")
	}

	function setupComparisonView() {
		for (let k = 3; k < numClasses+3; ++k){
			let average_value = {};
			average_value['quality'] = k
			instances.forEach(instance => {
				if(instance['Id'] == k) {
					for (let key in instance) {
						if (key !== 'Id' && key !== 'quality'){
							average_value[key] = 0
						}
					}
				}
			})


			instances.forEach(instance => {
				if(instance['quality'] == k) {
					for (let key in instance) {
						if (key !== 'Id' && key !== 'quality'){
							// console.log("here: ", instance['Id'], key, instance[key], )
							average_value[key] += Number(instance[key])
						}
					}
				}
			})

			for (let i in average_value) {
				if (i !== 'quality') {
					let num = average_value[i] / wineQualities[k]['instances'].length
					average_value[i] = Math.round((num + Number.EPSILON) * 100) / 100
				}
			}
			comparison_values.push(average_value)
		}
	}

	onMount(async () => {
		const fetched = await fetch("static/Wines.json");
		instances = (await fetched.json()).data;

		const fetched2 = await fetch("static/minmax.json");
		minMax = (await fetched2.json())

		const fetched3 = await fetch("static/correlation.json");
		correlation_dict = (await fetched3.json())
		console.log("correlation: ", correlation_dict)

		console.log(minMax)
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
        setupComparisonView()
        comparison_values = comparison_values
        console.log("Comparison Values: ", comparison_values)

		setupParallelCoordinates()

		yScale = scaleLinear().range([0, 500])
		yScaleTicks = yScale.ticks(5)

		xScale = scaleLinear().range([0, 10])
		xScaleTicks = xScale.ticks(1)

		featureScale = scalePoint().domain(features).range([20, 250]);

		chartSpread = scaleLinear().domain([0, 10]).range([0, 800]);
		chartSpreadTicks = chartSpread.ticks(11)

		for (let i = 3; i < numClasses+3; ++i) {
			let options = {};
			options['value'] = 'q'+[i]
			options['label'] = 'Quality'+[i]
			items.push(options)
		}
		items = items
		console.log("Dropdown options: ", items)

		for (let i in correlation_dict['quality']) {
			corr_array.push(correlation_dict['quality'][i])
		}
		corr_array = corr_array
		radar_arr = radar_arr

		corrColorScheme = scaleLinear().domain([ Math.min(...corr_array), 0, Math.max(...corr_array) ]).range(["red", "#ddd", "blue"]);
	});
// -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

	function radarClick(corr_val, feature_name, x) {

		if (radar_arr.includes(corr_val)) {
			radar_arr.pop(corr_val)
			x.setAttribute('stroke-width', '2')
			x.setAttribute('stroke', 'gray')
		}
		else {
			radar_arr.push(corr_val)
			x.setAttribute('stroke-width', '5')
			x.setAttribute('stroke', 'green')
		}
		radar_arr = radar_arr

		if (radar_labels.includes(feature_name)) {
			radar_labels.pop(feature_name)
		}
		else {
			radar_labels.push(feature_name)
		}
		radar_labels = radar_labels

// -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
		function setupRadarChart(){
			var marksCanvas = document.getElementById("marksChart");
			if (radar_arr.length > 6) {
				new_radar_arr = radar_arr.slice(0, 6)
			}

			for (let i=0; i < 6; ++i) {
				let idx = corr_array.indexOf(radar_arr[i])
				console.log("idx: " ,idx)
			}

			var marksData = {
				labels: radar_labels.slice(0, 6),
				datasets: [{
					label: "Quality 8",
					backgroundColor: "rgba(165,15,21,0.5)",
					// data: [1, 0.5917909543, 0.9977424972, 0.4786131524, 0.9615133085, 1]
				}, {
					label: "Quality 7",
					backgroundColor: "rgba(222,45,38,0.5)",
					// data: [0.9480313834, 0.6196717882, 0.9986368143, 0.4566643279, 0.9684384252, 0.9592565098]
				}, {
					label: "Quality 6",
					backgroundColor: "rgba(251,106,74,0.5)",
					// data: [0.8788761964,0.7231826107,0.9991489043,0.5624469486,0.9764779577,0.700119336]
				}, {
					label: "Quality 5",
					backgroundColor: "rgba(252,146,114,0.5)",
					// data: [0.8185333654, 1, 0.9996387108, 0.6523924432, 0.9726158343, 0.6230601722]
				}, {
					label: "Quality 4",
					backgroundColor: "rgba(252,187,161,0.5)",
					// data: [0.848744594, 0.6413510818, 0.9990761098, 0.7845814179, 0.9951469788, 0.4452722985]
				}, {
					label: "Quality 3",
					backgroundColor: "rgba(254,229,217,0.8)",
					// data: [0.8231051906, 0.4405991789, 1, 1, 1, 0.4372159091]
				}]
			};

			var radarChart = new Chart(marksCanvas, {
				type: 'radar',
				data: marksData
			});
		}

		setupRadarChart()

	}

</script>

<main>
	<h1>Multivariate Data Analyzer</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="input-view" class="view-panel">
				<div class="view-title">Input View</div>
				<div id="input-view-content">
					<svg height="380" width="441">
						{#each features as label,i}
							<text x="10" y="{i*30+15}" width="80%" height="10">{label}</text>
							<foreignObject x="170" y="{i*30}" width="170" height="30">
									<input type="range" min="0" max="30" value="0" class="slider" id="myRange">
								</foreignObject>
								<text x="350" y="{i*30+20}" width="80%" height="10">0</text>
						{/each}
						<rect x="10" y="330" width="300" height="30" fill="red"></rect>
						<text x="90" y="350" width="300" height="30" fill="white">Predict Quality</text>
					</svg>

				</div>
			</div>
			<div id="radar-view" class="view-panel">
				<div class="view-title">Radar Chart</div>
				<canvas id="marksChart" width="600" height="400"></canvas>
			</div>
		</div>
		<div id="main-section" >
			<div id="parcoord-view" class="view-panel" style="width: 980px;">
				<div class="view-title">Parallel Coordinates</div>

				<input type="radio" id="all" name="fav_language" value="all" style="margin-left: 35px;" checked="checked" on:click={()=>{
					selectedToggle = 0
					document.getElementById("parallel").innerHTML = ""
					setupParallelCoordinates()
				}}> Show All Data Lines
				<input type="radio" id="avg" name="fav_language" value="avg" on:click={()=>{
					selectedToggle = 1
					document.getElementById("parallel").innerHTML = ""
					setupParallelCoordinates()
				}}> Show Average Lines Per Quality
				<div id="parallel" style="float: left;"></div>
				<div style="float: right;">
					<svg width=110 height=300>
						<text x="0" y="20" class="small">Filter Classes</text>
						{#each colors as color, i}
							<rect x=0 y={(i+1)*30} width=20 height=20 fill={color}></rect>
						{/each}
					</svg>
				</div>
			</div>
<!-- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -->

			<div id="comparison-view" class="view-panel" >
				<div class="view-title">Compare Wine Quality</div>
					<div class="view-barChart">
						<svg viewbox="0 0 980 250" >
							<g>
								<text x=20 y=50>Quality</text>
								{#each corr_array as corr_val, index}
								<g id="corr-{index}">
									<foreignObject x={chartSpread(index)+80} y=0 width="85" height="150" >
										<div x={(index+0.5)*100} y=15 style="font-size: 10px;">
											{features[index]}
										</div>
									</foreignObject>
									<rect
										x={chartSpread(index)+80}
										y=25
										width=80 height=40
										stroke-width = '2'
										stroke = 'gray'
										style="
											fill: {corrColorScheme(corr_val)};"
										on:click={radarClick(corr_val, features[index], this)}
									/>
									<text x={chartSpread(index)+100} y=50>{corr_val}</text>
								</g>
								{/each}
							</g>
						</svg>
					</div>
			</div>
		</div>
	</div>
</main>
<!--  -->

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
		margin-bottom: 1px;
		padding: 3px 4px 4px 4px;
	}
	.quality-selection {
		height: 250px;
		width: 110px;
		padding-top: 10px;
		padding-left: 10px;
	}
	.view-barChart {
		height: 250px;
		width: 980px;
		padding: 10 10 10 10;
	}
	#input-view-content {
		height: 380px;
	}
	#comp-view {
		font-size: 11px;
		width: 10px;
		overflow-wrap: break-word;
	}
	#axis {
		stroke: black;
	}
	#quality-text {
		border: 1px;
		border-style: solid;
		border-color: black;
		border-left: none;
	}

</style>
