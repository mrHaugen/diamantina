<script lang="ts">
	let calculationTemplate = [
		{
			id: 'populationSize',
			name: 'Population size',
			type: 'input',
			value: null
		},
		{
			id: 'populationSizeCalculated',
			name: 'yet another function',
			type: 'calculation',
			param1: 2,
			value: function () {
				return this.param1 * 1000;
			}
		},
		{
			id: 'averageIncome',
			name: 'Average income',
			type: 'input',
			value: null
		}
	];

	//	let JSONcalculationTemplate = '[{"id": "populationSize","name": "Population size","type": "input","value": null},{"id": "averageIncome","name":"Average income","type": "input","value": null}]'
	//	let calculationTemplate = JSON.parse(JSONcalculationTemplate);

	let calculations = [
		[
			{
				id: 'populationSize',
				name: 'Population size',
				type: 'input',
				value: '2'
			},
			{
				id: 'averageIncome',
				name: 'Average income',
				type: 'input',
				value: '4'
			},
			{
				id: 'populationSizeCalculated',
				name: 'Populations size calculated',
				type: 'calculation',
				param: 2,
				value: function () {
					return this.param * 9;
				}
			}
		]
	];

	function addNewYear() {
		calculations = [...calculations, deepCopy(calculationTemplate)];
	}

	function deepCopy(obj: any) {
		if (typeof obj !== 'object' || obj === null) {
			return obj;
		}

		if (obj instanceof Date) {
			return new Date(obj.getTime());
		}

		if (obj instanceof Array) {
			return obj.reduce((arr, item, i) => {
				arr[i] = deepCopy(item);
				return arr;
			}, []);
		}

		if (obj instanceof Object) {
			return Object.keys(obj).reduce((newObj: any, key) => {
				newObj[key] = deepCopy(obj[key]);
				return newObj;
			}, {});
		}
	}

	function calculateSum(input: []) {
		return input.reduce((a, c) => {
			if (typeof c.value === 'function') {
				return Number(a) + c.value();
			}
			if (typeof c.value === 'string') {
				return Number(a) + Number(c.value);
			}
		}, 0);
	}
</script>

<div class="space-y-2">
	<button
		class="rounded-md border-2 border-blue-100 bg-blue-50 p-4 text-blue-950 hover:border-blue-500 hover:bg-blue-300"
		on:click={() => addNewYear()}>Add new year</button
	>

	<div class="flex flex-row space-x-3">
		{#each calculations as year, i}
			<div class="rounded-md border-2 p-2 px-5">
				Year {i + 1}
				{#each year as calculation, index}
					<div class="flex items-center py-1">
						<div class="flex-grow">
							{calculation.name}:
						</div>
						{#if calculation.type === 'calculation'}
							<div>
								{calculation.value()}
							</div>
						{:else if calculation.type === 'input'}
							<input
								class="w-24 rounded-md border-2 p-1 pr-2 text-right"
								type="text"
								bind:value={calculation.value}
							/>
						{/if}
					</div>
				{/each}
				Sum year: {calculateSum(year)}
			</div>
		{/each}
	</div>
</div>
