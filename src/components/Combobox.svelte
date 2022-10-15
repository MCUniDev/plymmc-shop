<script lang="ts">
    import ChevronUpIcon from '../icons/ChevronUpIcon.svelte';
    import { clickoutside } from '@svelte-put/clickoutside';
    import CheckIcon from '../icons/CheckIcon.svelte';
    import fuzzysearch from 'fuzzysearch';

    export let options: string[];
    export let value: string = '';

    let query = '';
    let open = false;
    let selected = '';

    $: filteredOptions = filter(query, options);

    $: if (options) {
        select(options[0]);
    }

    function toggle(value?: boolean) {
        open = value ?? !open;
    }

    function select(option: string) {
        selected = option;
        value = option;
        query = option;
    }

    function exit() {
        toggle(false);
        select(filteredOptions[0] || value);
    }

    function keyDown(event: KeyboardEvent) {
        toggle(true);

        if (event.key == 'Enter') {
            select(filteredOptions[0] || value);
        }
    }

    function filter(query: string, options: string[]) {
        if (query.trim().length) {
            const needle = query.replace(/ /g, '_').toLowerCase();

            return options.filter((option) =>
                fuzzysearch(needle, option.toLowerCase()),
            );
        }

        return options;
    }
</script>

<div class="relative" use:clickoutside on:clickoutside={exit}>
    <input
        type="text"
        class="w-full rounded-md border border-gray-300 bg-white py-2 pl-3 pr-12 shadow-sm focus:border-green-500 focus:outline-none focus:ring-1 focus:ring-green-500 sm:text-sm"
        role="combobox"
        on:keydown={keyDown}
        on:focusout={exit}
        bind:value={query}
        aria-controls="options"
        aria-expanded="false" />

    <button
        type="button"
        class="absolute inset-y-0 right-0 flex items-center rounded-r-md px-2 focus:outline-none"
        on:click={() => toggle()}>
        <ChevronUpIcon />
    </button>

    {#if open}
        <ul
            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm"
            id="options"
            role="listbox">
            {#each filteredOptions as option}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <li
                    class="relative select-none py-2 pl-3 pr-9 text-gray-900 hover:bg-green-500 cursor-pointer group"
                    aria-selected={option == selected}
                    on:click={() => select(option)}
                    role="option"
                    tabindex="-1">
                    <span
                        class="block truncate group-hover:text-white"
                        class:font-medium={option == selected}>
                        {option}
                    </span>

                    {#if option == selected}
                        <span
                            class="absolute inset-y-0 right-0 flex items-center pr-4 text-green-600 group-hover:text-white">
                            <CheckIcon />
                        </span>
                    {/if}
                </li>
            {/each}
        </ul>
    {/if}
</div>
