<script>
  // 基本は https://qiita.com/mkin/items/4259a1a9a850917a12b5 を参考にしている
  // 並び替え処理は https://reffect.co.jp/svelte/svelte-table-re-order/ を参考にしている
  import { onMount } from "svelte";
  import { flip } from "svelte/animate";
  import { cubicInOut } from "svelte/easing";

  let inputDom = null;
  let title = '';
  let condition = null;
  let todoList = [
    {id: 0, title: 'Aをする',done: false, order: 0},
    {id: 1, title: 'Bをする',done: false, order: 1},
    {id: 2, title: 'Cをする',done: false, order: 2},
    {id: 3, title: 'Dをする',done: false, order: 3},
    {id: 4, title: 'Eをする',done: false, order: 4},
    {id: 5, title: 'Fをする',done: false, order: 5},
  ];
  let dragIndex = null;
  let isInputValid = false;

  // 初期表示時にフォーカスをインプットにする
  onMount(() => {
    init();
  });

  function init() {
    title = "";
    isInputValid = false;
    inputDom.focus()
  }

  function addTask() {
    let newTodo = {id: todoList.length, title: title, done: false, order: todoList.length};
    todoList = [...todoList, newTodo];
    init();
  }
  
  // $:つけることで、conditionが変化するたびにこの関数が実行される
  $: filterTodo = condition === null ? todoList : todoList.filter((todo) => todo.done === condition);

  const dragStart = (index) => { 
    // indexにはドラッグ中の要素のindexが渡される
    dragIndex = index;
  };

  const dragEnter = (index) => {
    // indexには通過中の要素のindexが渡される
    if (index === dragIndex) {
      return;
    }

    // ドラッグ中のTODOと通過中のTODOを取り出す
    const draggedTodo = todoList.find(todo => todo.id === filterTodo[dragIndex].id);
    const targetTodo = todoList.find(todo => todo.id === filterTodo[index].id);

    // orderを入れ替える(分割代入)
    [draggedTodo.order, targetTodo.order] = [targetTodo.order, draggedTodo.order];
 
    // orderで並び替える
    todoList = todoList.sort((a, b) => a.order - b.order);
    dragIndex = index;
  }

  const dragEnd = () => {
    console.log('drag end');
    dragIndex = null;
  }
</script>

<div class="container">
  <div class="header">
      <h1>TODO リスト</h1>
  </div>

  <div class="button-group">
    絞り込み：
    <button class="button {condition === null ? 'selected' : ''}" on:click={() => condition = null}>全て</button>
    <button class="button {condition === false ? 'selected' : ''}" on:click={() => condition = false}>未完了</button>
    <button class="button {condition === true ? 'selected' : ''}" on:click={() => condition = true}>完了</button>
  </div>

  <!-- bind:value={title}で、インプットの内容を変数titleに反映することができる -->
  <!-- bind:this={inputDom}で、インプットのDOM要素自体をinputDomに反映することができる -->
  <div class="input-group">
    <input 
      class="input-field" type="text" 
      bind:value={title} bind:this={inputDom} 
      on:input={() => isInputValid = title.trim() !== ''}
      on:keydown={e => e.key === 'Enter' && !e.isComposing && isInputValid? addTask() : ''}
    />
    <button class="button add-button" on:click={addTask} disabled={!isInputValid}>タスク追加</button>
  </div>

  <div>
    <ul>
      {#each filterTodo as todo, i (todo.id)}
        <li 
          draggable=true 
          on:dragstart={() => dragStart(i)} 
          on:dragenter={() => dragEnter(i)} 
          on:dragover|preventDefault 
          on:dragend={dragEnd}
          class={i === dragIndex ? 'dragging' : ''}
          animate:flip={{ delay: 0, duration: 200, easing: cubicInOut }}
        >
          <input class="checkbox" type="checkbox" bind:checked={todo.done} on:click={() => todo.done = !todo.done}>
          {todo.title}
        </li>
      {/each}
    </ul>
  </div>  
</div>

<style>
  :root {
    font-family: 'Arial', sans-serif;
    background-color: whitesmoke;
    margin: 0;
    padding: 0;
  }

  .container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    background-color: #fff;
    border-radius: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .header {
    text-align: center;
    margin-bottom: 20px;
  }

  .button-group {
    text-align: center;
    margin-bottom: 20px;
  }

  .button {
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  .button.selected {
    background-color: #007bff;
    color: #fff;
  }

  .input-group {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
  }

  .input-field {
    flex-grow: 1;
    padding: 10px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  }

  .add-button {
    margin-left: 10px;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px;
    background-color: #fff;
    border-radius: 5px;
    margin-bottom: 10px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    transition: background-color 0.3s ease-in-out;
    cursor: grab;
  }

  .dragging {
    background-color: #eee;
    transform: scale(1.05);
    transition: background-color 0.3s ease-in-out, transform 0.2s ease-in-out;
  }

  input[type="checkbox"] {
    margin-right: 10px;
  }
</style>

