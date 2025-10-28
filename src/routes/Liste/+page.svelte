<svelte:head>
	<title>Génerateur de Liste</title>
</svelte:head>

<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let newList = '';
  let lists = {}; // toutes les listes

  // Charger les listes depuis le localStorage
  onMount(() => {
    const stored = localStorage.getItem('lists');
    if (stored) {
      lists = JSON.parse(stored);
    }
  });

  // Créer une nouvelle liste
  function createList() {
    if (!newList || lists[newList]) return; // pas de doublon ou vide
    lists = { ...lists, [newList]: [] };
    localStorage.setItem('lists', JSON.stringify(lists));
    goto(`/Liste/${newList}`);
    newList = '';
  }

  // Supprimer une liste
  function deleteList(name) {
    const confirmed = confirm(`Supprimer la liste "${name}" ?`);
    if (!confirmed) return;

    const { [name]: _, ...rest } = lists; // retirer la clé
    lists = rest;
    localStorage.setItem('lists', JSON.stringify(lists));
  }
</script>

<header>
    <a href="/">Retour</a>
</header>

<h1>Génerateur de Liste</h1>

<input bind:value={newList} placeholder="Nom de votre liste"/>
<button on:click={createList}>Créer la liste</button>

{#if Object.keys(lists).length == 0}
    <h2>Vous n'avez pas encore de liste</h2>
{:else}
    <h2>Mes Listes:</h2>
{/if}
<ul>
  {#each Object.keys(lists) as listName}
    <li>
      <a href={`/Liste/${listName}`}>{listName}</a>
      <button on:click={() => deleteList(listName)}>Supprimer</button>
    </li>
  {/each}
</ul>