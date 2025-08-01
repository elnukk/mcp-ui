<script>
  let district = "";
  let loading = false;
  let response = null;
  let error = null;
  let csvData = "";

  const districts = ["Arwal", "Aurangabad", "Banka", "Begusarai", "Bhagalpur", "Bhojpur", "Buxar", "Darbhanga", "East Champaran", "Gaya", "Gopalganj","Jamui","Jehanabad","Kaimur","Katihar","Khagaria","Kishanganj","Lakhisarai","Madhepura","Madhubani","Munger","Muzaffarpur","Nalanda","Nawada","Patna","Purnia","Rohtas","Saharsa","Samastipur","Saran","Sheikhpura","Sheohar","Sitamarhi","Siwan","Supaul", "Vaishali","West Champaran"];

  async function submit() {
    if (!district) return;
    loading = true;
    error = null;
    response = null;
    csvData = "";

    try {
      const res = await fetch("https://efarm.digitalgreen.org/agri_mcp", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ state: "bihar", district }),
      });

      if (!res.ok) {
        throw new Error(`API error: ${res.statusText}`);
      }

      const data = await res.json();
      response = data;
      csvData = data.csv || "";
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  function downloadCSV() {
    const blob = new Blob([csvData], { type: "text/csv" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = `${district.toLowerCase()}-workflow.csv`;
    link.click();
    URL.revokeObjectURL(url);
  }
</script>

<div class="container">
  <h1>BIHAR AgMCP</h1>
  <h2>Weather-based Crop Alerts</h2>
  
  <div class="form-section">
    <select bind:value={district}>
      <option value="" disabled selected>Select district</option>
      {#each districts as d}
        <option value={d}>{d}</option>
      {/each}
    </select>
    
    <button on:click={submit} disabled={!district || loading}>
      {loading ? "Running..." : "Run Workflow"}
    </button>
  </div>
  
  {#if response}
    <div class="response-section">
      <h2>Workflow Results</h2>
      {#if response.status === "success"}
        <div class="success-output">
          <pre>{response.message}</pre>
          {#if csvData}
            <button on:click={downloadCSV} style="margin-top: 10px;">
              Download CSV
            </button>
          {/if}
        </div>
      {:else}
        <div class="error-output">
          <pre>{response.message}</pre>
        </div>
      {/if}
    </div>
  {/if}
  
  {#if error}
    <div class="error-section">
      <p>Error: {error}</p>
    </div>
  {/if}
</div>

<style>
  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }

  h1 {
    color: #2c3e50;
    text-align: center;
  }
  h2 {
    color: #2c3e50;
    text-align: center;
  }
  h3 {
    color: #2c3e50;
    text-align: center;
  }

  .form-section {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    align-items: center;
    justify-content: center;
  }

  select {
    padding: 10px;
    border: 2px solid #ddd;
    border-radius: 5px;
    font-size: 16px;
    min-width: 200px;
  }

  button {
    padding: 10px 20px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
  }

  button:disabled {
    background-color: #bdc3c7;
    cursor: not-allowed;
  }

  button:hover:not(:disabled) {
    background-color: #2980b9;
  }

  .response-section {
    margin-top: 20px;
  }

  .success-output {
    background-color: #f8f9fa;
    border: 2px solid #28a745;
    border-radius: 5px;
    padding: 15px;
  }

  .error-output {
    background-color: #f8f9fa;
    border: 2px solid #dc3545;
    border-radius: 5px;
    padding: 15px;
  }

  .error-section {
    background-color: #f8d7da;
    border: 1px solid #f5c6cb;
    border-radius: 5px;
    padding: 10px;
    margin-top: 20px;
    color: #721c24;
  }

  pre {
    white-space: pre-wrap;
    word-wrap: break-word;
    font-family: 'Courier New', monospace;
    font-size: 14px;
    line-height: 1.4;
    margin: 0;
  }

  h2 {
    color: #2c3e50;
    border-bottom: 2px solid #3498db;
    padding-bottom: 5px;
  }
</style>
