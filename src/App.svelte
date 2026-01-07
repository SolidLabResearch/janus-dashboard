<script lang="ts">
    import SimpleQuery from "./lib/Query.svelte";
    import StreamChart from "./lib/StreamChart.svelte";

    let isRunning = true;
    let query = `REGISTER STREAM output AS
SELECT ?s ?val
FROM NAMED STREAM <stream> [RANGE 10s STEP 1s]
WHERE {
    ?s :hasValue ?val .
    FILTER(?val > 50)
}`;

    function handleQueryChange(newVal: string) {
        query = newVal;
    }

    function handleRegisterQuery() {
        console.log("Registering query:", query);
        // In a real app, this would send the query to the backend
        alert("Query Registered! (Check console for details)");
    }
</script>

<main>
    <aside>
        <div class="brand">Janus<span class="highlight"></span></div>

        <div class="editor-container">
            <SimpleQuery value={query} onChange={handleQueryChange} />
        </div>

        <div class="controls">
            <button class="register" on:click={handleRegisterQuery}>
                REGISTER QUERY
            </button>

            <button
                class:stop={isRunning}
                on:click={() => (isRunning = !isRunning)}
            >
                {isRunning ? "STOP PROCESSING" : "START STREAM"}
            </button>
        </div>
    </aside>

    <section>
        <StreamChart {isRunning} {query} />
    </section>
</main>

<style>
    :global(body) {
        margin: 0;
        font-family: "Inter", sans-serif;
        background: #f8f9fe;
        color: #333;
        overflow: hidden;
    }

    main {
        display: flex;
        height: 100vh;
        min-width: 1024px; /* Force desktop width */
        overflow-x: auto; /* Allow scrolling if window is smaller */
    }

    aside {
        width: 350px;
        background: #ffffff;
        border-right: 1px solid #e0e0e0;
        padding: 20px;
        display: flex;
        flex-direction: column;
        gap: 20px;
        box-shadow: 2px 0 5px rgba(0,0,0,0.02);
    }

    section {
        flex: 1;
        position: relative;
        background: #f8f9fe;
        display: flex;
        flex-direction: column;
        padding: 20px;
        min-width: 600px; /* Ensure space for chart */
    }

    .brand {
        font-size: 1.5rem;
        font-weight: 800;
        color: #6200ea; /* Deep Purple */
        margin-bottom: 10px;
    }
    .highlight {
        color: #2196f3; /* Blue */
    }

    /* Stream Descriptor Cards - REMOVED */


    .editor-container {
        flex: 1;
    }

    .controls {
        display: flex;
        flex-direction: column;
        gap: 12px;
    }

    button {
        width: 100%;
        padding: 16px;
        background: #6200ea; /* Deep Purple */
        color: white;
        font-weight: 600;
        border: none;
        border-radius: 12px;
        cursor: pointer;
        font-size: 1rem;
        transition: all 0.2s;
        box-shadow: 0 4px 12px rgba(98, 0, 234, 0.2);
    }

    button.register {
        background: #ffffff;
        color: #6200ea;
        border: 2px solid #6200ea;
        box-shadow: none;
    }
    button.register:hover {
        background: #f3e5f5;
        transform: translateY(-1px);
    }

    button:hover {
        background: #7c4dff;
        transform: translateY(-1px);
        box-shadow: 0 6px 16px rgba(98, 0, 234, 0.3);
    }
    button.stop {
        background: #ff1744;
        color: white;
        box-shadow: 0 4px 12px rgba(255, 23, 68, 0.2);
    }
    button.stop:hover {
        background: #d50000;
    }
</style>
