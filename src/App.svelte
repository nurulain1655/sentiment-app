<script>
    import UploadPage from "./UploadPage.svelte";
    import ResultPage from "./ResultPage.svelte";

    let page = "upload";
    let resultData = null; // Holds the full dictionary from Flask /upload
    let aiInsight = "";
    let loading = false;

    function goResult(data) {
        resultData = data;
        page = "result";
    }

    function goBack() {
        page = "upload";
        resultData = null;
        aiInsight = "";
    }

    async function getLlamaInsight(sampleText) {
        loading = true;
        aiInsight = ""; 
        
        try {
            const res = await fetch("http://127.0.0.1:5000/analyze-stigma", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                // Sending 'text' key because app.py uses data.get('text')
                body: JSON.stringify({ text: sampleText })
            });

            if (!res.ok) throw new Error("Flask server insight generation failed");

            const payload = await res.json();
            aiInsight = payload.insight; // Captures {"insight": "..."} safely
            
        } catch (error) {
            console.error("Error fetching Llama insight:", error);
            aiInsight = "Failed to load automated discourse insights. Make sure 'ollama run llama3:8b' is running in your terminal!";
        } finally {
            loading = false;
        }
    }
</script>

{#if page === "upload"}
    <UploadPage {goResult} />
{/if}

{#if page === "result"}
    <ResultPage 
        result={resultData} 
        {aiInsight} 
        {loading} 
        {getLlamaInsight} 
        {goBack} 
    />
{/if}