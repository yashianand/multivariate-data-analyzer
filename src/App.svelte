<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	let instances;

	const numClasses = 10;
	const numBins = 10;
	let binsByClasses = [];

	onMount(async () => {
		const fetched = await fetch("static/prediction_results.json");
		instances = (await fetched.json()).test_instances;
		console.log(instances);

		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}
		console.log(binsByClasses);

		instances.forEach(instance => {
			

		});

		

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