<script setup>
	import Chart from "chart.js/auto";
	import { ref, shallowRef, computed, watch, nextTick } from "vue";

	const localStorageKey = "weightData";
	const weights = ref([]);
	const weightChartElem = ref(null);
	const weightChart = shallowRef(null);
	const weightInput = ref();
	const weightNow = computed(() => {
		return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
	});

	// Function to save weight data to local storage
	const saveWeightDataToLocalStorage = () => {
		localStorage.setItem(localStorageKey, JSON.stringify(weights.value));
	};

	// Function to load weight data from local storage
	const loadWeightDataFromLocalStorage = () => {
		const storedData = localStorage.getItem(localStorageKey);
		if (storedData) {
			weights.value = JSON.parse(storedData);
		}
	};

	// Add initial data from local storage
	loadWeightDataFromLocalStorage();

	const addWeightInfo = () => {
		weights.value.push({
			weight: weightInput.value,
			date: new Date().getTime(),
		});

		// Save the updated data to local storage
		saveWeightDataToLocalStorage();
	};

	watch(
		weights,
		(newWeights) => {
			const ws = [...newWeights];

			if (weightChart.value) {
				weightChart.value.data.labels = ws
					.sort((a, b) => a.date - b.date)
					.map((w) => new Date(w.date).toLocaleDateString())
					.slice(-7);

				weightChart.value.data.datasets[0].data = ws
					.sort((a, b) => a.date - b.date)
					.map((w) => w.weight)
					.slice(-7);

				weightChart.value.update();
				return;
			}
			nextTick(() => {
				weightChart.value = new Chart(weightChartElem.value.getContext("2d"), {
					type: "line",
					data: {
						labels: ws.sort((a, b) => a.date - b.date).map((w) => new Date(w.date).toLocaleTimeString()),
						datasets: [
							{
								label: "weight",
								data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
								backgroundColor: "rgb(119,136,153)",
								borderColor: "rgb(255,255,255)",
								borderWidth: 1,
								fill: true,
							},
						],
					},
					options: {
						responsive: true,
						maintainAspectRatio: false,
					},
				});
			});
		},
		{ deep: true }
	);
</script>

<template>
	<main>
		<h1>Control your weight</h1>

		<div class="current">
			<span class="weight-number">{{ weightNow.weight }} kg</span>
			<span class="weight-number-info">your current weight</span>
		</div>

		<form @submit.prevent="addWeightInfo">
			<input required type="number" step="0.1" v-model="weightInput" placeholder="Type your weight here" />
			<button type="submit" placeholder="Add info about weight">Update</button>
		</form>

		<div v-if="weights && weights.length > 0">
			<h2>Last week</h2>
			<div class="canvas-area">
				<canvas ref="weightChartElem"></canvas>
			</div>

			<div class="weight-prev-info">
				<h3 class="prev-info-heading">Previous info about weight</h3>
				<ul>
					<li v-for="weight in weights">
						<span class="prev-weight-num">{{ weight.weight }}kg</span>
						<br />
						<span class="prev-weight-date">{{ new Date(weight.date).toLocaleDateString() }}</span>
					</li>
				</ul>
			</div>
		</div>
	</main>
</template>

<style scoped>
	main {
		padding: 1.5em;
	}
	ul {
		max-width: 600px;
		list-style: none;
		border: 1px solid gray;
		padding: 0.5em;
		border-radius: 10px;
		min-width: 50%;
		margin: auto;
	}
	h1 {
		font-size: 2em;
		text-align: center;
		margin-bottom: 2rem;
		text-transform: uppercase;
	}

	h2 {
		margin-bottom: 1rem;
		color: black;
		text-align: center;
		text-transform: uppercase;
	}
	h3 {
		text-align: center;
		font-size: 2em;
		text-transform: uppercase;
	}

	.prev-info-heading {
		margin-bottom: 2rem;
	}
	form {
		max-width: 500px;
		display: flex;
		flex-direction: column;
		margin: 0 auto 2rem auto;
	}
	.current {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		align-items: center;
		width: 150px;
		height: 150px;
		border-radius: 25%;
		background-color: rgba(0, 0, 0, 0.2);
		border: 5px solid black;
		margin: 0 auto 2rem;
		box-shadow: 0 4px 12px black;
	}

	.prev-weight-num {
		font-size: 1.5rem;
		font-weight: 700;
	}

	.prev-weight-date {
		font-size: 1.3rem;
		font-weight: 500;
	}
	.weight-number {
		font-size: 24px;
		font-weight: 700;
		text-align: center;
	}

	.weight-prev-info {
		max-width: 600px;
		margin: auto;
	}
	.canvas-area {
		margin-bottom: 2em;
		max-width: 600px;
		margin: auto;
	}

	form input[type="number"] {
		appearance: none;
		outline: none;
		border: none;
		background-color: whitesmoke;
		flex: 1 1 0%;
		padding: 1rem 1.5rem;
		font-size: 1.25rem;
		text-align: center;
	}

	button[type="submit"] {
		appearance: none;
		outline: none;
		border: none;
		cursor: pointer;
		padding: 1rem 1.5rem;
		flex: 1 1 0%;
		border: none;
		outline: none;
		font-size: 1.25rem;
		background-color: #c0c0c0;
		text-transform: uppercase;
	}

	li {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0.5rem;
		cursor: pointer;
	}

	li:nth-child(even) {
		background-color: rgba(0, 0, 0, 0.2);
	}

	.weight-number-info {
		text-align: center;
	}
</style>
