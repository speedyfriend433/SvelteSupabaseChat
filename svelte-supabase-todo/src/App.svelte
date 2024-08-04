<script>
  import { onMount } from 'svelte';
  import { supabase } from './supabase';

  let todos = [];
  let newTodo = '';

  async function fetchTodos() {
    let { data: todos, error } = await supabase
      .from('todos')
      .select('*')
      .order('id', { ascending: true });
    if (error) {
      console.error(error);
    } else {
      $todos = todos;
    }
  }

  async function addTodo() {
    if (newTodo.trim() !== '') {
      let { data, error } = await supabase
        .from('todos')
        .insert([{ text: newTodo }]);
      if (error) {
        console.error(error);
      } else {
        newTodo = '';
        fetchTodos();
      }
    }
  }

  async function toggleComplete(id, currentStatus) {
    let { data, error } = await supabase
      .from('todos')
      .update({ completed: !currentStatus })
      .eq('id', id);
    if (error) {
      console.error(error);
    } else {
      fetchTodos();
    }
  }

  async function removeTodo(id) {
    let { data, error } = await supabase
      .from('todos')
      .delete()
      .eq('id', id);
    if (error) {
      console.error(error);
    } else {
      fetchTodos();
    }
  }

  onMount(fetchTodos);
</script>

<style>
  .completed {
    text-decoration: line-through;
  }
</style>

<h1>Collaborative To-Do List</h1>

<input type="text" bind:value={newTodo} on:keypress="{e => e.key === 'Enter' && addTodo()}" />
<button on:click={addTodo}>Add</button>

<ul>
  {#each todos as todo (todo.id)}
    <li>
      <input type="checkbox" checked={todo.completed} on:change={() => toggleComplete(todo.id, todo.completed)} />
      <span class:completed={todo.completed}>{todo.text}</span>
      <button on:click={() => removeTodo(todo.id)}>Remove</button>
    </li>
  {/each}
</ul>
