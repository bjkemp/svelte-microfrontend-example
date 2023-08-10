<script lang="ts">
  import { get_current_component } from "svelte/internal"

  export let preview // If preview==='true', display dummy data for itemdetail contents
  export let dataid // must be present unless preview===true
  export let baseapiurl = "" // Not required for AEM, but is for Salesforce
  export let theme = "light" // 'light' or 'dark'

  type Data = {
    id: string
    name: string
    description: string
    first_name: string
    last_name: string
  }[];

  let data: Promise<Data>
  let elWidth
  let currentBreakpoint

  const hostname = window.location.hostname
  let getApiEndpoint

  if (hostname === 'localhost')
    getApiEndpoint = (baseUrl, id) => `json/is-svelte.json`
  else
    getApiEndpoint = (baseUrl, id) => `/admin/TE_Tech_Svelte/is-svelte.json`

  const getData = async (baseUrl): Promise<data> => {
    return await fetch(getApiEndpoint(baseUrl, dataid)).then((response) => {
      if (response.ok) {
        return response.json().then((json) => {
          data = json
        });
      } else {
        console.log("Network response was not ok.");
        throw Error(response.statusText);
      }
    });
  };

  const component = get_current_component();

  const success = () => {
    const event = new CustomEvent("itemdetailSpaSuccess", {
      detail: { testing: "testing" },
      bubbles: false,
      cancelable: true,
      composed: true, // makes the event jump shadow DOM boundary
    });

    component.dispatchEvent(event);
  };

  $: {
    if (typeof preview !== "undefined") {
      data = getData(baseapiurl)
    }
  }

  const breakpoints = {
    "bp-medium-up": 765,
    "bp-large-up": 1280,
  };

  $: {
    currentBreakpoint = Object.entries(breakpoints)
      .reduce((list, [key, value]) => {
        const match = elWidth >= value ? key : "";
        return [...list, match];
      }, [])
      .join(" ");
  }
</script>

<style lang="scss">
  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }

  .background {
    --bg-color: transparent;
    --text-color: white;
    --primary-color: #002e6e;

    &.dark {
      --bg-color: #0a3056;
      --text-color: #333;
      --primary-color: #c0e2ec;
    }

    background-color: var(--bg-color);
    color: var(--text-color);
    padding: 20px;
  }

  .data {
    background-color: var(--primary-color);
    border: 1px solid;
    border-color: var(--text-color);
    padding: 10px;
    border-radius: 6px;
    margin: 0 auto;
    max-width: 900px;

    &__title {
      margin-bottom: 20px;
      font-size: 23px;
    }
    &__description {
      font-size: 18px;
    }
  }

  .btn {
    width: 100%;
    border-radius: 6px;
    background-color: slategray;
    color: white;
    cursor: pointer;

    .bp-medium-up & {
      width: 100px;
    }
  }
</style>

<!--- Configure SPA ---->
<svelte:options tag="is-svelte" />
<div class="background {theme} {currentBreakpoint}" bind:clientWidth={elWidth}>
  {#await data}
    <div class="data data--loading">...loading...</div>
  {:then data}
    {#if data}
      {#each Object.entries(data) as [key, student]}
        <div class="data">
          {student.first_name} {student.last_name}
          <hr>
          {#each Object.entries(student) as [k, v]}
            <p class="data__description">{k}: {v}</p>
          {/each}
        </div>
      {/each}
    {/if}
  {:catch error}
    <div class="data">
      <p class="data__description">{error}</p>
    </div>
  {/await}
</div>
