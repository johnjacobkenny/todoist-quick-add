<script lang="ts">
  import { onDestroy, onMount } from "svelte";
  import { appWindow } from "@tauri-apps/api/window";
  import { register, unregister } from "@tauri-apps/api/globalShortcut";

  let todoContent = "";
  let apiKey = "";
  let hasApiKeyBeenEntered = false;

  const shortCutKeyCombo = "Super+Alt+Q";

  onMount(async () => {
    await register(shortCutKeyCombo, async () => {
      await appWindow.show();
    });
  });

  onDestroy(async () => {
    await unregister(shortCutKeyCombo);
  });

  const onSubmitTodo = async (event) => {
    var myHeaders = new Headers();
    myHeaders.append("Authorization", "Bearer " + apiKey);
    myHeaders.append("Content-Type", "application/json");

    var raw = JSON.stringify({ content: todoContent });

    var requestOptions = {
      method: "POST",
      headers: myHeaders,
      body: raw,
    };

    appWindow.hide();

    fetch("https://api.todoist.com/rest/v2/tasks", requestOptions)
      .then((response) => response.text())
      .then(async (result) => {
        todoContent = "";
      })
      .catch((error) => console.log("error", error));
  };

  const onSubmitApiKey = (event) => {
    hasApiKeyBeenEntered = apiKey !== "";
    appWindow.hide();
  };
</script>

<main class="container">
  {#if hasApiKeyBeenEntered === true}
    <form on:submit|preventDefault={onSubmitTodo}>
      <div class="form-container">
        <!-- svelte-ignore a11y-autofocus -->
        <input
          autofocus
          type="text"
          name="todo"
          class="textbox"
          id="todo"
          bind:value={todoContent}
        />
        <input type="submit" class="button" value="Add Task" />
      </div>
    </form>
  {:else}
    <form on:submit|preventDefault={onSubmitApiKey}>
      <div class="form-container">
        <!-- svelte-ignore a11y-autofocus -->
        <input
          autofocus
          type="text"
          name="todo"
          class="textbox"
          id="todo"
          bind:value={apiKey}
        />
        <input type="submit" class="button" value="Set API Key" />
      </div>
    </form>
  {/if}
</main>

<style>
  form {
    height: 100%;
  }

  .form-container {
    box-sizing: border-box;
    height: 100%;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    text-align: center;
    padding: 16px;
  }

  .textbox {
    font-size: 22px;
    flex-grow: 1;
    height: 32px;
  }

  .button {
    height: 36px;
    margin-left: 16px;
  }
</style>
