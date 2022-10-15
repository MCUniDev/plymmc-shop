<script lang="ts">
    import { clickoutside } from '@svelte-put/clickoutside';
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

<div class="relative max-w-6xl mx-auto" use:clickoutside on:clickoutside={exit}>
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
        <!-- Heroicon name: mini/chevron-up-down -->
        <svg
            class="h-5 w-5 text-gray-400"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true">
            <path
                fill-rule="evenodd"
                d="M10 3a.75.75 0 01.55.24l3.25 3.5a.75.75 0 11-1.1 1.02L10 4.852 7.3 7.76a.75.75 0 01-1.1-1.02l3.25-3.5A.75.75 0 0110 3zm-3.76 9.2a.75.75 0 011.06.04l2.7 2.908 2.7-2.908a.75.75 0 111.1 1.02l-3.25 3.5a.75.75 0 01-1.1 0l-3.25-3.5a.75.75 0 01.04-1.06z"
                clip-rule="evenodd" />
        </svg>
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
                            <!-- Heroicon name: mini/check -->
                            <svg
                                class="h-5 w-5"
                                xmlns="http://www.w3.org/2000/svg"
                                viewBox="0 0 20 20"
                                fill="currentColor"
                                aria-hidden="true">
                                <path
                                    fill-rule="evenodd"
                                    d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                                    clip-rule="evenodd" />
                            </svg>
                        </span>
                    {/if}
                </li>
            {/each}
        </ul>
    {/if}
</div>
