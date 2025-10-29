<svelte:head>
	<title>List - {listName}</title>
</svelte:head>

<script>
  import { page } from '$app/stores';
  import { onMount } from 'svelte';
  import jsPDF from 'jspdf'

  let listName = '';
  let items = [];
  let newItem = '';

  $: listName = $page.params.list_name;

  onMount(() => {
    const stored = localStorage.getItem('lists');
    if (stored) {
      const allLists = JSON.parse(stored);
      items = allLists[listName] || [];
    }
  });

  /* add one element on the list */
  function addItem() {
    const new_element = newItem.trim()

    if (!new_element) 
        return;

    if (items.some(element => element.text.toLowerCase() === new_element.toLowerCase()))
    {
        alert("Cette élément existe deja dans votre liste");
        return;
    }
    items = [...items, { text: newItem, count: 1 }];
    saveItems();
    newItem = '';
  }

  function increment(index) {
    items[index].count += 1;
    saveItems();
  }

  /* decrement on element from the list*/
  function decrement(index) {
    if (items[index].count > 0) {
      items[index].count -= 1;
      saveItems();
    }
    if (items[index].count == 0) {
        removeItem(index);
    }
  }

  /*remove one item*/
  function removeItem(index) {
    items = items.filter((_, i) => i !== index);
    saveItems();
  }

  /* save the item on the data base*/
  function saveItems() {
    const stored = localStorage.getItem('lists');
    const allLists = stored ? JSON.parse(stored) : {};
    allLists[listName] = items;
    localStorage.setItem('lists', JSON.stringify(allLists));
  }

  function PDF_Format(){
    const document = new jsPDF();

    document.setFontSize(18);
    document.text(`Liste : ${listName}`, 10, 15);

    document.setFontSize(12);
    let axeY = 30;

    if (items.length === 0) {
        document.text("Aucun élément dans cette liste")
    } else {
        items.forEach((item, count) => {
            document.text(`-${item.text} x${item.count}`, 10, axeY);
            axeY += 10;
        });
    }
    document.save(`${listName}.pdf`);
  }

</script>

<header class="p-2 bg-sky-200">
    <a href="/Liste">Retour</a>
</header>

<h1 class="text-4xl p-3 font-bold text-indigo-700 text-center">La liste : {listName}</h1>
<div class="flex flex-col items-center gap-5 pt-3 pb-4
md:flex-row md:pt-5 md:pl-3">
  <input bind:value={newItem} type="text" placeholder="Ajouter une élement "
   class="border-2 border-indigo-300 w-5/6 text-center focus:outline-none focus:border-indigo-700
    md:w-2/6"/>
  <button on:click={addItem} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-1
          hover:bg-indigo-500 hover:shadow-xl w-9/10 md:p-0 md:w-1/6 lg:w-1/7">
    Ajouter
  </button>
</div>

{#if (items.length == 0)}
    <p class="text-center pt-5">il y pas d'élement dans votre liste</p>
{:else if items.length == 1}
    <p class="underline font-bold p-3">il y a {items.length} élement dans votre liste :</p>
{:else if items.length > 1}
    <p class="underline font-bold p-3">il y a {items.length} élements dans votre liste :</p>
{/if}

<ul>
  {#each items as item, i}
    <li class="flex flex-row justify-between m-3 p-3 gap-3 bg-indigo-300 rounded-2xl items-center
    ">
        <h2 class="font-bold">{item.text}</h2>
        <div class="flex flex-row items-center gap-2">
          <button on:click={() => decrement(i)}
            class="border-2 border-indigo-500 bg-indigo-200 text-xl pl-1 pr-1
          hover:bg-indigo-500 hover:shadow-xl text-center">
            -
          </button>
          <p>{item.count}</p>
          <button on:click={() => increment(i)}
            class="border-2 border-indigo-500 bg-indigo-200 text-xl pl-0.5 pr-0.5
          hover:bg-indigo-500 hover:shadow-xl text-center">
            +
          </button>
        </div> 
    </li>
  {/each}
</ul>

{#if (items.length > 0)}
  <button on:click={PDF_Format} class="border-2 border-indigo-500 bg-indigo-200 rounded-2xl p-1
  hover:bg-indigo-500 hover:shadow-xl w-9/10 md:p-0 md:w-1/6 lg:w-1/6">
    Exporter en PDF
  </button>
{/if}