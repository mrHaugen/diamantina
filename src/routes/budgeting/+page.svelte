<script lang="ts">
	import { deepCopy } from '$lib/tools';
	import { onMount } from 'svelte';

	let template: any = {
		populationSize: {
			label: 'Population size',
			value: 1000000
		},
		activePopulation: {
			label: 'Active population',
			value: null
		},
		averageIncomePerPerson: {
			label: 'Average income per person [DIP]',
			value: 3000000
		},
		propertyTax: {
			label: 'Property tax [DIP]',
			value: 2000
		},
		incomeTaxPercent: {
			label: 'Income tax [%]',
			value: 0.2
		},
		industryVATpercent: {
			label: 'Industry VAT [%]',
			value: 0.17
		},
		agricultureVATpercent: {
			label: 'Agriculture VAT [%]',
			value: 0.12
		},
		tradeVATpercent: {
			label: 'Trade VAT [%]',
			value: 0.15
		},
		industryVAT: {
			label: 'Industry VAT [M DIP]',
			value: 0
		},
		agricultureVAT: {
			label: 'Agriculture VAT [M DIP]',
			value: 0
		},
		tradeVAT: {
			label: 'Trade VAT [M DIP]',
			value: 0
		},
		totalIncome: {
			label: 'EGSD [M DIP]',
			value: 0
		},
		budgetArmy: {
			label: 'Army budget [DIP]',
			value: 990000000000
		},
		budgetPolice: {
			label: 'Police budget [DIP]',
			value: 990000000000
		},
		budgetHealthCare: {
			label: 'Health Care budget [DIP]',
			value: 1500000
		},
		budgetEducation: {
			label: 'Education budget [DIP]',
			value: 5000000
		},
		budgetJustice: {
			label: 'Justice budget [DIP]',
			value: 990000000000
		},
		budgetPensions: {
			label: 'Pension budget [DIP]',
			value: 13300000
		},
		budgetInvestments: {
			label: 'Investment budget [DIP]',
			value: 2600000000
		},
		budgetBalance: {
			label: 'Blance [M DIP]',
			value: 0
		}
	};

	let model: any = [];

	function updateModel(model: any) {
		rules.forEach((rule) => {
			rule.rule();
		});
	}

	$: {
		updateModel(model);
	}

	let rules = [
		{
			description: 'Population growth',
			rule: function () {
				model.forEach((year: any, index: number) => {
					if (index !== 0) {
						year.populationSize.value = Number(model[index - 1].populationSize.value) + 500000;
						year.populationSize.readOnly = true;
					} else {
						year.populationSize.value = year.populationSize.value;
					}
				});
			}
		},
		{
			description: 'Active population',
			rule: function () {
				model.forEach((year: any) => {
					year.activePopulation.value = Math.round(year.populationSize.value * 0.66);
					year.activePopulation.readOnly = true;
				});
			}
		},
		{
			description: 'Warning Max tax before Revolution ',
			rule: function () {
				model.forEach((year: any) => {
					year.incomeTaxPercent.value > 0.2
						? (year.incomeTaxPercent.warning = 'Tax to high: might cause Revolution.')
						: (year.incomeTaxPercent.warning = '');
				});
			}
		},
		{
			description: 'VAT from Industry',
			rule: function () {
				model.forEach((year: any) => {
					year.industryVAT.value = Math.round(
						(year.activePopulation.value * year.industryVATpercent.value * 2e7) / 1e6
					);
					year.industryVAT.readOnly = true;
				});
			}
		},
		{
			description: 'VAT from agriculture',
			rule: function () {
				model.forEach((year: any) => {
					year.agricultureVAT.value = Math.round(
						(year.activePopulation.value * year.agricultureVATpercent.value * 1e7) / 1e6
					);
					year.agricultureVAT.readOnly = true;
				});
			}
		},
		{
			description: 'VAT from trade',
			rule: function () {
				model.forEach((year: any) => {
					year.tradeVAT.value = Math.round(
						(year.activePopulation.value * year.tradeVATpercent.value * 3e7) / 1e6
					);
					year.tradeVAT.readOnly = true;
				});
			}
		},
		{
			description: 'Total income aka EGSD, utput in [kDIP]',
			rule: function () {
				model.forEach((year: any) => {
					let incomeTaxMDIP =
						(year.activePopulation.value *
							year.averageIncomePerPerson.value *
							year.incomeTaxPercent.value) /
						1e6;
					let propertyTaxMDIP = (year.activePopulation.value * year.propertyTax.value) / 1e6;
					let industryVATMDIP = year.industryVAT.value;
					let agricultureVATMDIP = year.agricultureVAT.value;
					let tradeVATMDIP = year.tradeVAT.value;

					let EGSDMDIP =
						incomeTaxMDIP + propertyTaxMDIP + industryVATMDIP + agricultureVATMDIP + tradeVATMDIP;
					year.totalIncome.value = Math.round(EGSDMDIP);
					year.totalIncome.readOnly = true;
				});
			}
		},
		{
			description: 'Budget balance in [M DIP]',
			rule: function () {
				model.forEach((year: any) => {
					let expensesMDIP =
						(Number(year.budgetArmy.value) +
							Number(year.budgetPolice.value) +
							Number(year.budgetHealthCare.value) +
							Number(year.budgetEducation.value) +
							Number(year.budgetJustice.value) +
							Number(year.budgetPensions.value) +
							Number(year.budgetInvestments.value)) /
						1e6;
					let balanceMDIP = year.totalIncome.value - expensesMDIP;

					year.budgetBalance.value = Math.round(balanceMDIP);
					year.budgetBalance.readOnly = true;
				});
			}
		}
	];

	function addNewYear() {
		model = [...model, deepCopy(template)];
	}

	onMount(async () => {
		model = [...model, deepCopy(template)];
		model = [...model, deepCopy(template)];
	});
</script>

<div class="flex flex-row">
	<div class="flex shrink-0 flex-col p-3">
		<div class="h-10 font-bold">What</div>
		{#each Object.entries(template) as field}
			<div class="flex h-12 items-center">
				{field[1].label}
			</div>
		{/each}
	</div>

	{#each model as year, indexYear}
		<div class="flex flex-col p-3">
			<div class="h-10 pr-3 text-right font-bold">
				Year {indexYear + 1}
			</div>

			<div class="flex flex-row">
				<div>
					{#each Object.keys(year) as field, indexField}
						<div class="flex h-12 items-center">
							{#if year[field].readOnly}
								<span class="w-36 p-2 text-right">{year[field].value}</span>
							{:else}
								<input
									type="text"
									class="w-36 rounded-md border-2 p-2 text-right"
									class:border-red-500={year[field].warning}
									bind:value={year[field].value}
								/>
							{/if}
						</div>
						<div class="w-36 text-xs text-red-700">
							{year[field].warning ? year[field].warning : ''}
						</div>
					{/each}
				</div>
			</div>
		</div>
	{/each}
	<div class="h-12 shrink-0">
		<button
			class="rounded-md border-2 border-blue-300 bg-blue-50 p-4 hover:border-blue-800 hover:bg-blue-200"
			on:click={() => addNewYear()}>Add new year</button
		>
	</div>
</div>
