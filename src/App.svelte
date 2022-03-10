<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleSequential, scalePoint } from "d3-scale";
	import { create, all } from 'mathjs'

	const config = { }
	const math = create(all, config)
	let instances;
	let wineQualities = {};
	let features = ["alcohol", "total sulfur dioxide", "density", "volatile acidity", "pH", "citric acid", "fixed acidity", "residual sugar", "chlorides", "free sulfur dioxide", "sulphates", "quality"]
	let radar_labels = [];
	let minMax;
	let correlation_dict = {};
	let corr_array = [];
	let selectedToggle = 0 //it can be 0, 1 or 2
	let comparison_values = [];
	let items = [];
	let corrColorScheme;
	let xScale, xScaleTicks, yScale, yScaleTicks;
	let colors = ["#A50F15", "#DE2D26", "#FB6A4A", "#FC9272", "#FCBBA1", "#FEE5D9", "blue"]
	let chartSpread, chartSpreadTicks, featureScale;
	$: filteredClasses = [] //by default we are showing all the classes
	const numClasses = 6;
	let key, value;
	let radar_arr = [];
	let new_radar_arr = [];
	console.log(radar_arr)
	let slider_values = [];
	let userEnteredSample = {
		"Id": "NA",
		"fixed acidity": "4.6",
		"volatile acidity": "0.12",
		"citric acid": "0",
		"residual sugar": "0.9",
		"chlorides": "0.012",
		"free sulfur dioxide": "1",
		"total sulfur dioxide": "6",
		"density": "0.99007",
		"pH": "2.74",
		"sulphates": "0.33",
		"alcohol": "8.4",
		"quality": "2"
	}



	function changeSlider(value, x){
		let slider_id = value.target.id
		let slider_value = value.target.value
		var output = document.getElementById("output" + slider_id.slice(6));
		output.innerHTML = slider_value;
	}

	function predictPressed(){
		// console.log('PREDICT PRESSED!')
		for( const label in features) {
			let id = "slider-" + features[label]
			var slider = document.getElementById(id)
			let value= slider.value.toString();
			console.log("hello")
			userEnteredSample[features[label]] = value
			slider_values.push(value)
			slider_values = slider_values
		}
		console.log('ENTERED SAMPLE: ', userEnteredSample)
		document.getElementById("parallel").innerHTML = ""
		setupParallelCoordinates()
	}

	function convertJsonToArray(json_var){
		let result = []

		for(var i in json_var){
    		result.push(json_var[i]);
		}
		result.shift();
		result.splice(-1)
		return result
	}

	function calculateDistance(a, b){
		var array_a = convertJsonToArray(a).map(x => parseFloat(x))
		var array_b = convertJsonToArray(b).map(x => parseFloat(x))

		let result = math.distance(array_a, array_b)
		return result
	}

	function calculateKNearest(k){

		let distanceToAll = []
		for (const ins in instances) {
			let distance = calculateDistance(userEnteredSample, instances[ins])
			let json_dist = instances[ins]
			json_dist["distance"] = distance
			// console.log("json_dist: ", json_dist)
			distanceToAll.push(json_dist)
			// console.log("distanceToAll: ", distanceToAll)
		}
		distanceToAll.sort(compare)

		console.log("yeeee1", instances)

		distanceToAll.map(x => delete x['distance'])
		console.log("yeeee2", instances)


		if (k!==0){
			return distanceToAll.slice(0, k)
		}
		return distanceToAll
	}
	let showRadar;

	function compare(a,b) {
		if ( a.distance < b.distance ){
			return -1;
		}
		if ( a.distance > b.distance ){
			return 1;
		}
		return 0;
	}

	function setupParallelCoordinates(){
		// set the features and margins of the graph
		const margin = {top: 10, right: 5, bottom: 20, left: 30},
		width = 850 - margin.left - margin.right,
		height = 430 - margin.top - margin.bottom;

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
            data = instances.filter(function (sample){
                return !filteredClasses.includes(parseInt(sample.quality))
            });
        } else if (selectedToggle == 1){
            data = comparison_values
        } else {
            let  k = 10
            data = calculateKNearest(k)
            let sum = 0
            for(const dddd in data){
                sum += parseInt(data[dddd].quality)
            }
            userEnteredSample.quality = (sum/k).toString()
            data.push(userEnteredSample)
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
			average_value['quality'] = k.toString()
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

	function filterClass(selectedClass){
		filteredClasses.includes(selectedClass)? filteredClasses.splice(filteredClasses.indexOf(selectedClass), 1) : filteredClasses.push(selectedClass)
		console.log(filteredClasses)
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
        setupComparisonView()
        comparison_values = comparison_values
        console.log("Comparison Values: ", comparison_values)

        yScale = scaleLinear().domain([0, value]).range([0, 100])
		yScaleTicks = yScale.ticks(5)

		xScale = scaleLinear().range([0, 50])
		xScaleTicks = xScale.ticks(2)
		console.log('filtered classes', filteredClasses)
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
			if (radar_arr.length < 6) {
				radar_arr.push(corr_val)
				x.setAttribute('stroke-width', '5')
				x.setAttribute('stroke', 'green')
			}
		}
		radar_arr = radar_arr

		if (radar_labels.includes(feature_name)) {
			radar_labels.pop(feature_name)
		}
		else {
			if (radar_labels.length < 6) {
				radar_labels.push(feature_name)
			}
		}
		radar_labels = radar_labels

		function setupRadarChart(){
			let radar_colors = ["#FEE5D9", "#FCBBA1", "#FC9272", "#FB6A4A",  "#DE2D26", "#A50F15"]
			var marksCanvas = document.getElementById("marksChart");
			// normalization
			let norm = {}
			for (let i = 0; i < comparison_values.length; ++i) {
				for (const [feature, feature_value] of Object.entries(comparison_values[i])) {
					if (feature !== 'quality') {
						norm[feature] = 0
					}
				}
			}
			for (let i = 0; i < comparison_values.length; ++i) {
				for (const [feature, feature_value] of Object.entries(comparison_values[i])) {
					if (feature !== 'quality') {
						if (feature_value > norm[feature]){
							norm[feature] = feature_value
						}
					}
				}
			}
			console.log("norm: ", norm)

			for (let i=0; i < 6; ++i) {
				showRadar = i
			}
			let final_dataset_array = []
			for (let i=0; i < comparison_values.length; ++i) {
				let quality_dict = {
					'label': 'Quality'+[i+3],
					'backgroundColor': "rgba(165,15,21,0)",
					'borderColor': radar_colors[i],
					'data': []
				}
				for (const [feature, feature_value] of Object.entries(comparison_values[i])) {
					for (let j = 0; j < radar_labels.length; ++j) {
						if (radar_labels[j] == feature) {
							let norm_feature_value = feature_value / norm[feature]
							quality_dict['data'].unshift(norm_feature_value)
							console.log(quality_dict['data'])
						}
					}
				}
				final_dataset_array.push(quality_dict)
			}

			var marksData = {
				labels: radar_labels.slice(0, 6),
				datasets:  final_dataset_array
			};
			console.log("dataset: ", marksData)


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
		<div id="sidebar" style="width: 450px; height: 500px;">
			<div id="input-view" class="view-panel">
				<div class="view-title">Input View</div>
				<div id="input-view-content">
					<svg height="500" width="441">
						{#each features as label,i}
							<text x="15" y="{i*37+20}" width="80%" height="10">{label}</text>
							{#if slider_values !== undefined}
								<text id={"output-" + label} x="350" y="{i*37+25}" width="80%" height="10">0</text>

							{/if}
							<foreignObject x="170" y="{i*37}" width="170" height="30">
								{#if minMax !== undefined}
									<input type="range" min={minMax[label].Min} max={minMax[label].Max} value="0" class="slider" id={"slider-" + label} on:input={
									changeSlider}>
									<!-- <output> -->
								{/if}
							</foreignObject>
						{/each}
						<rect x="10" y="410" width="400" height="30" fill="red" on:click={()=>{ predictPressed() }}></rect>
						<text x="160" y="430" width="400" height="30" fill="white" on:click={()=>{ predictPressed() }}>Predict Quality</text>
					</svg>

				</div>
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
				<input type="radio" id="knn" name="fav_language" value="knn" on:click={()=>{
					selectedToggle = 2
					document.getElementById("parallel").innerHTML = ""
					setupParallelCoordinates()
				}}> Nearest Wine Samples To Entered Sample
				<!-- {#if userEnteredSample !== undefined}
				{/if} -->

				<div id="parallel" style="float: left;"></div>
				<div style="float: right;">
					<svg width=110 height=300>
						<text x="0" y="20" class="small">Filter Qualities</text>
						{#each colors as color, i}
							<rect x=0 y={(i+1)*30} width=20 height=20 fill={color} ></rect>
							{#if filteredClasses}
								<foreignObject x="4" y={(i+1)*30} width="160" height="160">
									<input type="checkbox" id="vehicle1" name="vehicle1" value="Bike" style="background-color: red" checked="checked" on:click={()=>{
										console.log('i: ', i)
										filterClass(8-i)
										document.getElementById("parallel").innerHTML = ""
										setupParallelCoordinates();
									}}> Quality {8-i}
								</foreignObject>
							{/if}
						{/each}
					</svg>
				</div>
			</div>
		</div>
	</div>
	<div id="container" class="view-bottom-panel" style="width: 1500px;">
		<div id="title">

		</div>
		<div id="sidebar" class="view-divider" style="width: 450px; height: 400px;">
			<div class="view-title">Radar Chart</div>
			<div id="radar-view">
				{#if showRadar == undefined}
					<p id="radar-text">Select features to show radar chart</p>
				{/if}
				<canvas id="marksChart" width="200" height="160"></canvas>
			</div>
		</div>
		<div class="view-Corr" id="main-section" style="width: 1000px;">
			<div class="view-title">Compare Wine Quality</div>
			<svg height=400>
				<g>
					<text x=930 y=210>Quality</text>
					{#each corr_array as corr_val, index}
						<g id="corr-{index}">
							<foreignObject x={chartSpread(index)+20} y=120 width="75" height="15" >
								<div x={(index+0.5)*100} y=105 style="font-size: 10px;">
									{features[index]}
								</div>
							</foreignObject>
							<rect
								x={chartSpread(index)+10}
								y=140
								width=70 height=130
								stroke-width = '2'
								stroke = 'gray'
								style="
									fill: {corrColorScheme(corr_val)};"
								on:click={radarClick(corr_val, features[index], this)}
							/>
							<text x={chartSpread(index)+30} y=210>{corr_val}</text>
						</g>
					{/each}
				</g>
			</svg>
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
	#sidebar, #main-section, #title {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
		height: 100%;
		width: 98%;
	}
	.view-divider {
		border-right: 2px solid #eee;
		margin-right: 0px;
		height: 100%;
	}
	.view-bottom-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
		width: 100%;

	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 1px;
		padding: 3px 4px 4px 4px;
	}
	#input-view-content {
		height: 380px;
	}
	#radar-text {
		font-size: medium;
		color: gray;
		text-align: center;
	}




</style>
