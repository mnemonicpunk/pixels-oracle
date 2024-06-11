<script>
    import { createEventDispatcher } from 'svelte';
  
    export let data;
  
    // Define the config object
    const config = {
      'id': (value) => `<h1>${value}</h1>`,
      'categories': value => {
        let cards = "";
        value.map(item => {
            cards += `<div class="viewer_card_tag">${item}</div>`;
        });
        return `<div class="viewer_card"><b>Categories:</b>${cards}</div>`
      },
      'image': value => `<img src=${value}>`,  
      'sprite': value => `<img src=${value.image} style="width: ${value.width}px, height: ${value.height}px, ">`,
      'tier': value => `<b>Tier:</b> Test`,
      'craftable': (value) => `
        <p>Crafting (${value.minutesRequired} minute${value.minutesRequired > 1 ? 's' : ''}, ${value.result.energyUsage.value} energy):<ul>
        ${value.requiredItems.map(item => `<li>${item.quantity}x ${item.id}</li>`).join('\n')}
        </ul></p>`
      // Add more custom rendering functions for other keys as needed
    };
  
    // Display mode variable with initial value
    $: displayMode = 'Display';
  
    // Create a dispatcher for custom events
    const dispatch = createEventDispatcher();
  
    // Function to handle radio button click and update display mode
    const handleDisplayModeChange = (event) => {
      displayMode = event.target.value;
      dispatch('modeChange', displayMode); // Dispatch a custom event for reactivity
      data = data;
    };
  
    // Function to render a specific key based on display mode
    const renderKey = (key, value) => {
      if (displayMode === 'Raw') {
        return "<b>" + key + "</b>" + JSON.stringify(value);
      } else {
        if (config[key] && typeof config[key] === 'function') {
          return config[key](value);
        }
      }
    };
  
    // Default rendering method
    const defaultRender = (key, value) => {
      if (typeof value === 'object' && value !== null) {
        return Object.keys(value).map(subKey => renderKey(subKey, value[subKey])).join('\n');
      } else {
        return `${key}: ${value}`;
      }
    };
  </script>
  
  <div>
    <div>
      Display:
      <input type="radio" name="displayMode" value="Display" on:change={handleDisplayModeChange} checked={displayMode === 'Display'} />
    </div>
    <div>
      Raw:
      <input type="radio" name="displayMode" value="Raw" on:change={handleDisplayModeChange} checked={displayMode === 'Raw'} />
    </div>
    {#each Object.keys(data) as key}
      {#if displayMode == "Raw"}
        <div class="json_entry">
            <b>{key}</b> {JSON.stringify(data[key])}
        </div>
      {:else}
        {#if key=="id"}
            <h1>{data[key]}</h1>
        {/if}
        {#if key=="image"}
            <img src={data[key]} alt={key}>
        {/if}
        {#if key=="sprite"}
            <img src={data[key].image} alt={key}>
        {/if}
        {#if key=="categories"}
            <div class="viewer_card">
                <b>Categories:</b>
                {#each Object.values(data[key]) as item}
                    <div class="viewer_card_tag">
                        {item}
                    </div>
                {/each}
            </div>
        {/if}
        {#if key=="craftable"}
          {#if data[key].requiredSkill}
            <div class="viewer_card">
              Requires <b>{data[key].requiredSkill}{data[key].requiredLevel?" "+data[key].requiredLevel + " ":""}</b>
            </div>
          {/if}
          <div class="viewer_card">
              Crafting ({data[key].minutesRequired} minute{data[key].minutesRequired > 1 ? 's' : ''}, {data[key].result?.energyUsage?.value} energy):<br>
              <ul>
                  {#each Object.values(data[key].requiredItems) as item}
                      <li>{item.quantity}x {item.id}</li>
                  {/each}
              </ul>
          </div>
        {/if}
        {#if key=="items"}
            <div class="viewer_card">   
                {#each Object.values(data[key]) as item}
                    <div class="viewer_card_list_item">   
                        <b>{item.item}:</b>
                        {#if item.buyPrice}
                            <div class="viewer_card_tag{(item.buyStatus == 'lock'|| item.buyStatus == 'hide'|| item.buyStatus === null)?' locked_tag':''}"> 
                                Buy: {item.buyPrice || "n/a"} <img style="width: 12px; height: 12px" src={item.currency=="cur_coins"?"//d31ss916pli4td.cloudfront.net/uploadedAssets/currency/cur_coins/abf1e7f3-994d-46d0-a7d1-a0bdbef2c4b1.png":"//d31ss916pli4td.cloudfront.net/uploadedAssets/currency/pixels_pixel_currency_new_2.png"} alt="currency">
                            </div>
                        {/if}
                        {#if item.sellPrice}
                            <div class="viewer_card_tag"> 
                                Sell: {item.sellPrice || "n/a"} <img style="width: 12px; height: 12px" src={item.currency=="cur_coins"?"//d31ss916pli4td.cloudfront.net/uploadedAssets/currency/cur_coins/abf1e7f3-994d-46d0-a7d1-a0bdbef2c4b1.png":"//d31ss916pli4td.cloudfront.net/uploadedAssets/currency/pixels_pixel_currency_new_2.png"} alt="currency">
                            </div>
                        {/if}
                    </div>
                {/each}
            </div>
        {/if}
        {#if key=="tier"}
            <div class="viewer_card">
              <b>Tier:</b> {data[key]}
            </div>
        {/if}
        {#if key=="inventory"}
            <div class="viewer_card">
              {#if data[key].maxQuantity}
                <b>Stack size:</b> {data[key].maxQuantity}
              {/if}
            </div>
        {/if}
      {/if}
    {/each}
    {#if ((data.kind == "music") || (data.kind == "fx")) && (data.url)}
        <audio class="viewer_audio_player" src={data.url} controls />
    {/if}
  </div>
  
  <style>
    /* Styles for the GameDataViewer component */
    .viewer_card {
        margin-bottom: 22px;
    }

    .viewer_card_tag {
        padding: 11px;
        background: #448;
        border-radius: 22px;
        display: inline;
        margin-left: 3px;
        margin-right: 3px;
    }

    .viewer_audio_player {
        width: 100%;
        height: 44px;
    }

    .viewer_card_list_item {
        padding: 11px;
    }

    .locked_tag {
      background: #444;
      color: #666;
      text-decoration: line-through;
    }
  </style>
  