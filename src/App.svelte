<script lang="ts">
    import RefreshIcon from './icons/RefreshIcon.svelte';
    import Combobox from './components/Combobox.svelte';
    import Chart from './components/Chart.svelte';
    import type { RawData, Data } from './types';
    import csv from 'csvtojson';

    let selected: string;
    let promise = loadData();

    async function loadData(): Promise<Data> {
        const res = await fetch('https://upmc-shop-csv.deno.dev');

        if (!res.ok) {
            throw new Error('Response not OK');
        }

        const csvData = await res.text();
        const result: RawData = await csv().fromString(csvData);

        // This is an object of an item (e.g. cake) to an array of it's prices
        const data: Record<string, number[]> = {};

        for (const row of result) {
            // Loop over every row in the csv and merge the data
            for (const [key, value] of Object.entries(row)) {
                // fix weird bug
                if (key == 'field462') continue;

                if (!data[key]) {
                    data[key] = [];
                }

                if (value.length) {
                    data[key].push(+value);
                }
            }
        }

        return Object.entries(data);
    }
</script>

<main class="space-y-6 max-w-7xl mx-auto px-4">
    <div class="my-4 text-center space-y-2">
        <img class="w-96 mx-auto" src="/UOPMC.webp" alt="UOPMC Logo" />
        <h1 class="font-medium text-2xl">Survival Shop Price History</h1>
    </div>

    {#await promise}
        <p class="text-center font-medium">Loading...</p>
    {:then data}
        <div class="max-w-6xl mx-auto w-full">
            <Combobox options={data.map((d) => d[0])} bind:value={selected} />
        </div>

        {#if selected}
            <Chart point={data.find((d) => d[0] == selected)} />
        {/if}

        <div class="max-w-6xl mx-auto w-full">
            <button
                on:click={() => (promise = loadData())}
                class="block ml-auto rounded-md border transition-colors border-gray-300 px-3 py-2 text-sm hover:bg-green-500 hover:border-green-500 focus:outline-none focus:ring-2 hover:text-white focus:ring-green-500 focus:ring-offset-2">
                <RefreshIcon />
            </button>
        </div>
    {:catch error}
        <p class="text-center font-medium text-red-500">{error}</p>
    {/await}
</main>
