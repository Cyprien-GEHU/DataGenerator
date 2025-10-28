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
            document.text(`${count + 1}. ${item.text} - compteur : ${item.count}`, 10, axeY);
            axeY += 10;
        });
    }
    document.save(`${listName}.pdf`);
  }

</script>

<header>
    <a href="/Liste">Retour</a>
</header>

<h2>La liste : {listName}</h2>

<input bind:value={newItem} type="text" placeholder="Ajouter une élement "/>
<button on:click={addItem}>Ajouter</button>

{#if (items.length == 0)}
    <p>il y pas d'élement dans votre liste</p>
{:else if items.length > 0}
    <p>il y a {items.length} élement dans votre liste</p>

<ul>
  {#each items as item, i}
    <li>
        {item.text} 
        <button on:click={() => decrement(i)}>-</button>
        {item.count}
        <button on:click={() => increment(i)}>+</button>
      
    </li>
  {/each}
</ul>

<button on:click={PDF_Format}>Exporter en PDF</button>
{/if}