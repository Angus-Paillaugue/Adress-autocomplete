<script>
    import { onMount } from 'svelte';

    let supportsSpeech = false;
    let result = false;
    let searchResults = [];
    let searchQuery;
    let recognition;
    let isListening = false;


    $: search(), searchQuery;

    async function search() {
        if(!searchQuery) return;
        if(searchQuery == result?.properties?.label) return;
        if(searchQuery.length < 3) {searchResults = [];return};
        const res = await fetch(`https://api-adresse.data.gouv.fr/search/?q=${searchQuery}&limit=7`, { method: "GET", mode: "cors", "Access-Control-Allow-Origin":"*" });
        const adresses = await res.json();
        searchResults  = adresses.features;
    }

    onMount(() => {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        supportsSpeech = !!SpeechRecognition;
        if(SpeechRecognition) {
            recognition = new SpeechRecognition();
        }
    });

    function handleVoiceBtn() {
        if(isListening){
            recognition.abort();
            isListening = false;
        }else {
            try{
                recognition.continuous = true;
                recognition.lang = window.navigator.language;
                recognition.start();
                isListening = true;
                recognition.addEventListener("result", (event) => {
                    console.log(event)
                    const transcript = event.results[event.resultIndex][0].transcript.trim();
                    searchQuery = transcript;
                });
            }catch(_){}
        }
        
    }
</script>
<main class="min-h-screen flex flex-col justify-center items-center">

    <div class="max-w-xl w-full mx-auto">
        <label for="voice-search" class="sr-only">Search</label>
        <div class="relative w-full">
            <div class="relative w-full">
                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-4 h-4 stroke-2 text-gray-500 dark:text-gray-400"><path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z" /></svg>
                </div>
                <input type="text" id="voice-search" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm {searchResults.length == 0 ? "rounded-lg" : "focus:rounded-b-none rounded-lg"} transition-all focus:border-primary-300 block w-full pl-10 p-2.5  dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:border-primary-500 focus:ring-primary-300 outline-none ring-0 focus:ring-2 peer" autocomplete="off" placeholder="Rechercher une adresse" bind:value={searchQuery}>
                
                {#if supportsSpeech}
                    <button type="button" class="absolute inset-y-0 right-0 flex items-center text-gray-500 dark:text-gray-400 hover:text-gray-900 dark:hover:text-white bg-transparent hover:bg-gray-200 rounded-lg text-sm justify-center dark:hover:bg-gray-600 h-full aspect-square" on:click={handleVoiceBtn}>
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-4 h-4"><path stroke-linecap="round" stroke-linejoin="round" d="M12 18.75a6 6 0 006-6v-1.5m-6 7.5a6 6 0 01-6-6v-1.5m6 7.5v3.75m-3.75 0h7.5M12 15.75a3 3 0 01-3-3V4.5a3 3 0 116 0v8.25a3 3 0 01-3 3z" /></svg>
                    </button>
                {/if}
                {#if searchResults.length > 0}
                    <div class="absolute bottom-0 left-0 w-full rounded-b-lg bg-gray-50 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 translate-y-[calc(100%-2px)] flex-col flex {searchResults.length > 0 ? "peer-focus:opacity-100 peer-focus:z-50 pointer-events-auto" : ""} opacity-0 -z-10 transition-all">
                        {#each searchResults as adresse, index}
                            <button class="p-1 text-start block w-full {index + 1 == searchResults.length ? "rounded-b-lg": "border-b border-gray-300 dark:border-gray-600"}" on:click={() => {searchQuery = adresse.properties.label; result = adresse; searchResults = [];}}>
                                {adresse.properties.label}
                            </button>
                        {/each}
                    </div>
                {/if}
            </div>
        </div>
    </div>

</main>

{#if result}
    <section>
        <div class="max-w-xl w-full mx-auto">
            <iframe class="w-full aspect-video" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" src="https://maps.google.com/maps?q={result.geometry.coordinates.reverse().join(",")}&hl=fr&z=16&output=embed" title=""></iframe>
        </div>
    </section>
{/if}