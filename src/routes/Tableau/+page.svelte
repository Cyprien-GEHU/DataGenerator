<svelte:head>
	<title>Génerateur de Tableau</title>
</svelte:head>

<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let newTab = '';
  let Tables = {};

  // change the table from the localstorage
  onMount(() => {
    const stored = localStorage.getItem('Tables');
    if (stored) {
      Tables = JSON.parse(stored);
    }
  });

  // add a new table
  function createTab() {
    if (!newTab || Tables[newTab]) return;
    Tables = { ...Tables, [newTab]: [] };
    localStorage.setItem('Tables', JSON.stringify(Tables));
    goto(`/Tableau/${newTab}`);
    newTab = '';
  }

  // delete a Tables
  function deleteTab(name) {
    const confirmed = confirm(`Supprimer la Tables "${name}" ?`);
    if (!confirmed) return;

    const { [name]: _, ...rest } = Tables;
    Tables = rest;
    localStorage.setItem('Tables', JSON.stringify(Tables));
  }
</script>

<header class="p-2 bg-sky-200">
    <a href="/">Retour</a>
</header>

<h1 class="text-4xl p-3 font-bold text-indigo-700 text-center">Génerateur de tableau</h1>

<div class="flex flex-col items-center gap-5 pt-3 pb-4
md:flex-row md:pt-5 md:pl-3">
  <input bind:value={newTab} placeholder="Nom de votre Tableau"
    class="border-2 border-indigo-300 w-5/6 text-center focus:outline-none focus:border-indigo-700
    md:w-2/6"/>
  <button on:click={createTab} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-1
          hover:bg-indigo-500 hover:shadow-xl w-9/10 md:p-0 md:w-1/6 lg:w-1/7">
    Créer le tableau
  </button>
</div>

{#if Object.keys(Tables).length == 0}
    <h2 class="text-center pt-5">Vous n'avez pas encore de tableau !</h2>
{:else}
    <h2 class="underline font-bold p-3">Mes tableaux:</h2>
{/if}
<ul class="flex flex-col items-center flex-wrap justify-center sm:flex-row lg:justify-evenly">
  {#each Object.keys(Tables) as tabName}
    <li class="flex flex-col items-center m-3 p-3 gap-3 bg-indigo-300 rounded-2xl w-70 sm:w-4/10 lg:w-1/5
    ">
      <h3 class="font-bold">{tabName}</h3>
      <a href={`/Tableau/${tabName}`} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl w-3/4
        hover:bg-indigo-500 hover:shadow-xl text-center"
        >Consulté le tableau</a>
      <button on:click={() => deleteTab(tabName)} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl w-3/4
        hover:bg-indigo-500 hover:shadow-xl">
        Supprimer le tableau
      </button>
    </li>
  {/each}
</ul>

