<script lang="ts">
    import type { RawData, Data } from './types';
    import csv from 'csvtojson';
    import Chart from './Chart.svelte';
    import Combobox from './Combobox.svelte';

    let selected: string;

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

    {#await loadData()}
        <p class="text-center font-medium">Loading...</p>
    {:then data}
        <div class="w-full h-full space-y-4">
            <Combobox options={data.map((d) => d[0])} bind:value={selected} />

            {#if selected}
                <Chart point={data.find((d) => d[0] == selected)} />
            {/if}
        </div>
    {:catch error}
        <p class="text-center font-medium text-red-500">{error}</p>
    {/await}
</main>
