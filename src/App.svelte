<script>
  let phrase = '';
  let meaning = '';
  let loading = false;

  const apikey = import.meta.env.VITE_COHERE_API;

  const getMeaning = async () => {
    if (phrase === '') {
      meaning = 'Please enter a phrase';
      return;
    }
    loading = true;
    try {
      const response = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${phrase.trim()}`);
      const data = await response.json();
      meaning = data[0].meanings[0].definitions[0].definition;
    } catch (error) {
      try {
        const response = await fetch('https://api.cohere.ai/v1/generate', {
          headers: {
            Authorization: 'BEARER ' + apikey,
            'Content-Type': 'application/json',
          },
          method: 'POST',
          body: JSON.stringify({
            model: 'command-xlarge-nightly',
            prompt: `Write what is the meaning of ${phrase.trim()}?`,
            max_tokens: 100,
            temperature: 0.2,
            k: 0,
            stop_sequences: [],
            return_likelihoods: 'NONE',
          }),
        });
        const data = await response.json();
        meaning = data.generations[0].text;
      } catch (error) {
        meaning = 'Something went wrong';
      }
    } finally {
      loading = false;
    }
  };

  const handleInput = (event) => {
    phrase = event.target.value;
  };
</script>

<main class="flex flex-col items-center w-full min-h-screen p-4 pb-12 mx-auto bg-slate-900">
  <h1
    class="mt-32 text-4xl font-bold text-center text-transparent bg-clip-text bg-gradient-to-r from-green-300 via-blue-500 to-purple-600"
  >
    PhraseAI
  </h1>
  <div class="flex flex-col items-center justify-center w-full max-w-sm mt-4">
    <textarea
      class="w-full p-4 text-lg rounded-md shadow-md appearance-none resize-none focus:outline-none bg-slate-800 text-slate-200"
      name=""
      id=""
      rows="3"
      placeholder="Type a phrase..."
      on:input={handleInput}
    />
    <button
      on:click={getMeaning}
      class="px-4 py-2 mt-4 bg-indigo-500 rounded-md shadow-md text-slate-200 shadow-indigo-900">Get meaning</button
    >
    {#if loading}
      <p class="mt-8 text-slate-200">Loading...</p>
    {:else}
      <p class="mt-8 text-slate-200">{meaning}</p>
    {/if}
  </div>
</main>
