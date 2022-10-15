<script lang="ts">
    import type { RawData, Data } from './types';
    import Combobox from './Combobox.svelte';
    import Chart from './Chart.svelte';
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
                <!-- Heroicon name: mini/arrow-path -->
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                    class="w-5 h-5">
                    <path
                        fill-rule="evenodd"
                        d="M15.312 11.424a5.5 5.5 0 01-9.201 2.466l-.312-.311h2.433a.75.75 0 000-1.5H3.989a.75.75 0 00-.75.75v4.242a.75.75 0 001.5 0v-2.43l.31.31a7 7 0 0011.712-3.138.75.75 0 00-1.449-.39zm1.23-3.723a.75.75 0 00.219-.53V2.929a.75.75 0 00-1.5 0V5.36l-.31-.31A7 7 0 003.239 8.188a.75.75 0 101.448.389A5.5 5.5 0 0113.89 6.11l.311.31h-2.432a.75.75 0 000 1.5h4.243a.75.75 0 00.53-.219z"
                        clip-rule="evenodd" />
                </svg>
            </button>
        </div>
    {:catch error}
        <p class="text-center font-medium text-red-500">{error}</p>
    {/await}
</main>
