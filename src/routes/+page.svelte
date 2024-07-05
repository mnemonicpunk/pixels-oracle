<script>
  import { onMount } from "svelte";
  import GameDataViewer from "./GameDataViewer.svelte";

  let jsonData = null;
  let groups = [];
  let selectedKey = null;
  let searchQuery = "";
  let jsonLoaded = false; // New variable to track if JSON data is loaded

  const loadJson = (event) => {
    const files = event.dataTransfer
      ? event.dataTransfer.files
      : event.target.files;

    if (!files.length) {
      console.error("No file selected.");
      return;
    }

    const file = files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        jsonData = JSON.parse(e.target.result);
        groups = Object.keys(jsonData).map((group) => ({
          name: group,
          keys: Object.keys(jsonData[group]),
          count: Object.keys(jsonData[group]).length,
          open: false,
        }));
        jsonLoaded = true; // Set jsonLoaded to true
      } catch (error) {
        console.error("Error parsing JSON file:", error);
      }
    };
    reader.readAsText(file);
  };

  const selectKey = (groupName, key) => {
    selectedKey = jsonData[groupName] ? getDataByKey(groupName, key) : null;
    console.log(selectedKey);
  };

  const getDataByKey = (groupName, key) => {
    if (jsonData && jsonData[groupName]) {
      const nestedData = jsonData[groupName][key];
      return nestedData !== undefined ? nestedData : "Data not found";
    }
    return "Data not found";
  };

  const onSearch = (event) => {
    const sanitizedQuery = event.target.value.toLowerCase().trim();
    searchQuery = sanitizedQuery;
  };

  function toggleGroup(groupName) {
    groups = groups.map((group) => {
      if (group.name === groupName) {
        group.open = !group.open;
      }
      return group;
    });
  }

  function displayJsonData(key) {
    selectedKey = key;
  }

  // Function to fetch the recent game library
  async function fetchRecentGameLib() {
    const baseAPIURL = 'https://pixels-server.pixels.xyz/v1';
    const tenant = 'pixels';

    // Fetch the current version information
    const versionResponse = await fetch('https://play.pixels.xyz/version.json');
    const versionData = await versionResponse.text();
    const clientVersion = versionData.trim();

    // Check if the version is correctly fetched
    if (!clientVersion) {
      throw new Error('Version not found in the version data');
    }

    // Function to hash the version string
    function hashVersion(version) {
      const versionMap = {
        "6.122": "87bbbwei20",
        "6.209": "--78DEVO+spins"
      };

      if (versionMap[version]) {
        return versionMap[version];
      }

      let hash = 0;
      for (let i = 0; i < version.length; i++) {
        const charCode = version.charCodeAt(i) + i - 17;
        hash = (hash << 5) - hash + charCode;
        hash |= 0; // Convert to 32bit integer
      }
      return hash;
    }

    // Set up the headers with the hashed client version
    const headers = {
      'x-client-version': hashVersion(clientVersion)
    };

    // Fetch the game library
    const response = await fetch(`${baseAPIURL}/game/library?tenant=${tenant}&ver=${clientVersion}`, {
      headers: headers
    });

    // Parse the JSON response
    const gameLibrary = await response.json();
    return gameLibrary;
  }

  // Handler to fetch the prod game library and update jsonData
  async function handleFetchProdGameLib() {
    try {
      jsonData = await fetchRecentGameLib();
      groups = Object.keys(jsonData).map((group) => ({
        name: group,
        keys: Object.keys(jsonData[group]),
        count: Object.keys(jsonData[group]).length,
        open: false,
      }));
      jsonLoaded = true; // Set jsonLoaded to true
    } catch (error) {
      console.error("Error fetching game library:", error);
      alert("Error fetching game lib:\n" + error +"\n\nMay need to use a CORS unblock extension.");
    }
  }

  onMount(() => {
    // Add event listeners for drag and drop
    const dropZone = document.getElementById("drop-zone");

    dropZone.addEventListener("dragover", (e) => {
      e.preventDefault();
      dropZone.classList.add("drag-over");
    });

    dropZone.addEventListener("dragleave", () => {
      dropZone.classList.remove("drag-over");
    });

    dropZone.addEventListener("drop", (e) => {
      e.preventDefault();
      dropZone.classList.remove("drag-over");
      loadJson(e);
    });
  });
</script>

<div id="container">
  <div id="list">
    {#if !jsonLoaded}
      <div id="drop-zone">
        <p>Drag & drop a JSON file here</p>
        <input
          type="file"
          accept=".json"
          style="display: none;"
          on:change={loadJson}
        />
      </div>
      <div style="padding: 22px;">- OR -</div>
      <button on:click={handleFetchProdGameLib}>Fetch prod game lib</button>
    {/if}

    {#if jsonData}
      <input type="text" placeholder="Search keys..." on:input={onSearch} />

      {#each groups as group}
        <div class="group" class:zero-members={group.count === 0}>
          <p class="group-name" on:click={() => toggleGroup(group.name)}>
            {group.name} ({group.count})
          </p>
          {#if group.open}
            {@const filteredKeys = group.keys.filter((key) =>
              key.toLowerCase().includes(searchQuery),
            )}
            <ul class="keys">
              {#each filteredKeys as key}
                <li on:click={() => selectKey(group.name, key)}>{key}</li>
              {/each}
            </ul>
          {/if}
        </div>
      {/each}
    {/if}
  </div>

  <div id="viewer">
    {#if selectedKey}
      <div class="viewer-content">
        <GameDataViewer data={selectedKey} />
      </div>
    {/if}
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #151c3e, #0d1132);
    color: #ffffff;
  }

  #container {
    display: flex;
    height: 100vh;
  }

  #list {
    flex: 1;
    overflow: auto;
    padding-right: 10px;
  }

  #viewer {
    flex: 3;
    overflow-y: auto;
    padding-left: 10px;
  }

  #drop-zone {
    width: 100%;
    border: 2px dashed #aaa;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.5rem;
    color: #aaa;
  }

  .drag-over {
    background-color: #f0f0f0;
  }

  .group {
    margin-bottom: 10px;
  }

  .group-name {
    cursor: pointer;
  }

  .keys {
    margin-left: 10px;
  }

  .keys li {
    cursor: pointer;
  }

  .zero-members {
    color: #ccc;
  }

  .viewer {
    background-color: #1f2558;
    padding: 20px;
    border-left: 2px solid #3c4a96;
  }

  pre {
    margin: 0;
  }
</style>
