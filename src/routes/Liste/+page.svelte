<svelte:head>
	<title>Génerateur de Liste</title>
</svelte:head>

<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let newList = '';
  let lists = {};

  // change the list from the localstorage
  onMount(() => {
    const stored = localStorage.getItem('lists');
    if (stored) {
      lists = JSON.parse(stored);
    }
  });

  // add a new list
  function createList() {
    if (!newList || lists[newList]) return; // pas de doublon ou vide
    lists = { ...lists, [newList]: [] };
    localStorage.setItem('lists', JSON.stringify(lists));
    goto(`/Liste/${newList}`);
    newList = '';
  }

  // delete a list
  function deleteList(name) {
    const confirmed = confirm(`Supprimer la liste "${name}" ?`);
    if (!confirmed) return;

    const { [name]: _, ...rest } = lists; // retirer la clé
    lists = rest;
    localStorage.setItem('lists', JSON.stringify(lists));
  }
</script>

<header class="p-2 bg-sky-200">
    <a href="/">Retour</a>
</header>

<h1 class="text-4xl p-3 font-bold text-indigo-700 text-center">Génerateur de Liste</h1>

<div class="flex flex-col items-center gap-5 pt-3 pb-4
md:flex-row md:pt-5 md:pl-3">
  <input bind:value={newList} placeholder="Nom de votre liste"
    class="border-2 border-indigo-300 w-5/6 text-center focus:outline-none focus:border-indigo-700
    md:w-2/6"/>
  <button on:click={createList} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-1
          hover:bg-indigo-500 hover:shadow-xl w-9/10 md:p-0 md:w-1/6 lg:w-1/7">
    Créer la liste
  </button>
</div>

{#if Object.keys(lists).length == 0}
    <h2 class="text-center pt-5">Vous n'avez pas encore de liste !</h2>
{:else}
    <h2 class="underline font-bold p-3">Mes Listes:</h2>
{/if}
<ul class="flex flex-col items-center flex-wrap justify-center sm:flex-row lg:justify-evenly">
  {#each Object.keys(lists) as listName}
    <li class="flex flex-col items-center m-3 p-3 gap-3 bg-indigo-300 rounded-2xl w-70 sm:w-4/10 lg:w-1/5
    ">
      <h3 class="font-bold">{listName}</h3>
      <a href={`/Liste/${listName}`} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl w-3/4
        hover:bg-indigo-500 hover:shadow-xl text-center"
        >Regarder la liste</a>
      <button on:click={() => deleteList(listName)} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl w-3/4
        hover:bg-indigo-500 hover:shadow-xl">
        Supprimer la liste
      </button>
    </li>
  {/each}
</ul>