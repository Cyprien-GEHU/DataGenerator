<script>
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  let tabName = '';
  let Tables = {};
  let tableData = [[]]; // tableau 2D : lignes et colonnes

  // récupérer le nom depuis l’URL
  $: tabName = $page.params.tab_name;

  onMount(() => {
    const stored = localStorage.getItem('Tables');
    if (stored) {
      Tables = JSON.parse(stored);
      if (Tables[tabName]) {
        tableData = Tables[tabName];
      } else {
        goto('/Tableau'); // ✅ redirection correcte
      }
    }
  });

  // sauvegarde dans le localStorage
  function saveTable() {
    Tables[tabName] = tableData;
    localStorage.setItem('Tables', JSON.stringify(Tables));
  }

  // ajouter une ligne
  function addRow() {
    const colCount = tableData[0]?.length || 1;
    const newRow = Array(colCount).fill('');
    tableData = [...tableData, newRow];
    saveTable();
  }

  // ajouter une colonne
  function addColumn() {
    tableData = tableData.map((row) => [...row, '']);
    saveTable();
  }

  // modifier une cellule
  function updateCell(rowIndex, colIndex, value) {
    tableData[rowIndex][colIndex] = value;
    saveTable();
  }

  // supprimer une ligne
  function deleteRow(index) {
    if (tableData.length <= 1) return;
    tableData = tableData.filter((_, i) => i !== index);
    saveTable();
  }

  // supprimer une colonne
  function deleteColumn(index) {
    if (tableData[0].length <= 1) return;
    tableData = tableData.map((row) => row.filter((_, i) => i !== index));
    saveTable();
  }

  // supprimer tout le tableau
  function deleteThisTable() {
    const confirmDelete = confirm(`Supprimer complètement le tableau "${tabName}" ?`);
    if (!confirmDelete) return;
    const { [tabName]: _, ...rest } = Tables;
    Tables = rest;
    localStorage.setItem('Tables', JSON.stringify(Tables));
    goto('/Tableau'); // ✅ retour vers /Tableau
  }
</script>

<!-- ✅ Header -->
<header class="p-2 bg-sky-200">
  <a href="/Tableau">⬅ Retour</a>
</header>

<h1 class="text-4xl p-3 font-bold text-indigo-700 text-center">
  Tableau : {tabName}
</h1>

<!-- ✅ Boutons d’action -->
<div class="flex flex-wrap justify-center gap-3 mt-4">
  <button
    on:click={addRow}
    class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-2 hover:bg-indigo-500 hover:shadow-xl"
  >
    ➕ Ajouter une ligne
  </button>
  <button
    on:click={addColumn}
    class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-2 hover:bg-indigo-500 hover:shadow-xl"
  >
    ➕ Ajouter une colonne
  </button>
  <button
    on:click={deleteThisTable}
    class="border-2 border-red-500 bg-red-200 rounded-2xl p-2 hover:bg-red-500 hover:shadow-xl"
  >
    🗑️ Supprimer ce tableau
  </button>
</div>

<!-- ✅ Le tableau -->
<div class="overflow-x-auto flex justify-center mt-6">
  <table class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl shadow-xl">
    <tbody>
      {#each tableData as row, rowIndex}
        <tr>
          {#each row as cell, colIndex}
            <td class="border-2 border-indigo-300 p-2">
              <input
                type="text"
                bind:value={tableData[rowIndex][colIndex]}
                on:input={(e) => updateCell(rowIndex, colIndex, e.target.value)}
                class="w-24 text-center border-2 border-indigo-200 focus:outline-none focus:border-indigo-700 rounded-md"
              />
            </td>
          {/each}
          <td>
            <button
              on:click={() => deleteRow(rowIndex)}
              class="ml-2 border-2 border-red-500 bg-red-200 rounded-2xl px-2 hover:bg-red-500 hover:shadow-xl"
            >
              delete line
            </button>
          </td>
        </tr>
      {/each}
      <tr>
        {#each tableData[0] as _, colIndex}
          <td class="text-center">
            <button
              on:click={() => deleteColumn(colIndex)}
              class="border-2 border-red-500 bg-red-200 rounded-2xl px-2 hover:bg-red-500 hover:shadow-xl"
            >
              delete col
            </button>
          </td>
        {/each}
      </tr>
    </tbody>
  </table>
</div>

<style>
  table {
    border-collapse: collapse;
    min-width: 300px;
  }
</style>
