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

<style>
  :global(body) {
    font-family: 'Open Sans', sans-serif;
    margin: 0;
    padding: 0;
  }

  .background {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-gap: 20px;
    padding: 20px;
    background-color: #f5f5f5;
  }

  .data {
    background-color: white;
    border-radius: 6px;
    padding: 20px;
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  }

  .data__name {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 10px;
  }

  .data__description {
    font-size: 16px;
    margin-bottom: 5px;
  }

  .data__key {
    font-weight: bold;
  }

  .btn {
    display: inline-block;
    padding: 10px 20px;
    border-radius: 6px;
    background-color: #007bff;
    color: white;
    text-decoration: none;
    transition: background-color 0.3s ease;
  }

  .btn:hover {
    background-color: #0069d9;
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
          {student.LAST_NAME}, {student.FIRST_NAME} {student.NICKNAME || ''}
          <hr>
          <div>
            Grade: {student.GRADE_LEVEL}, {student.STUDENT_NUMBER}
          </div>
          <div>
            Fall: {student.FALL_SPORTS || ''}
          </div>
          <div>
            Winter: {student.WINTER_SPORTS || ''}
          </div>
          <div>
            Spring: {student.SPRING_SPORTS || ''}
          </div>
          <hr>
          <textarea name="" id="" cols="30" rows="5"></textarea>
          {student.CONTACT_PHONE || ''}
          {student.PORTAL_EMAIL || ''}

          <button class="button">Email Guardian</button>
          <hr>
          <div>
            <label for="IMPACT">Impact</label>
            <input type="checkbox" name="IMPACT">
            <label for="IMPACT_DATE">Date</label>
            <input type="date" name="IMPACT_DATE">
          </div>


          {#each Object.entries(student.FORM_PACKET) as [k, v]}
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
