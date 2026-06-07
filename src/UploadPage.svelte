<script>
export let goResult;

let file = null;
let fileName = "Click to choose CSV..."; // Added to show the name once selected

function handleFile(event) {
    file = event.target.files[0];
    if (file) {
        fileName = file.name; // Updates the bar text to the file name
    }
}

async function upload() {
    if (!file) {
        alert("Please select a file first!");
        return;
    }

    const formData = new FormData();
    formData.append("file", file);

    try {
        const res = await fetch("http://127.0.0.1:5000/upload", {
            method: "POST",
            body: formData
        });

        if (!res.ok) throw new Error("Upload failed");

        const data = await res.json();
        goResult(data);
    } catch (error) {
        console.error("Error:", error);
        alert("Make sure your Flask server is running!");
    }
}
</script>

<style>

:global(body) {
    background-color: #f0f4f8; 
    margin: 0;
    font-family: 'Inter', sans-serif;
}

h1.main-header {
    text-align: center;
    background: #7ed957; /* Or your preferred green */
    color: white;
    padding: 20px;
    margin: 0;
    font-size: 32px;
}

.landing-page {
    display: flex;
    flex-direction: column; /* Stacks everything vertically */
    align-items: center; /* Centers everything horizontally */
    justify-content: center; /* Centers everything vertically on screen */
    gap: 40px; /* Space between the text and the box */
    padding: 20px 20px;
    min-height: calc(100vh - 80px); /* Fills the rest of the screen */
    max-width: 1200px;
    margin: auto;
    text-align: center; /* Centers the description text */
}

.info-section {
    max-width: 800px; /* Prevents text from stretching too wide */
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 0;
}

.info-section h2 {
    font-size: 3rem; 
    color: #2d3436;
    margin-bottom: 20px;
    margin-top: 0%;
}

.info-section p {
    font-size: 1.2rem;
    color: #636e72;
    line-height: 1.6;
    margin-bottom: 20px;
}

/* Stats (Below the description) */
.stats-mini {
    display: flex;
    gap: 20px;
    color: #4caf50;
    font-weight: bold;
    font-size: 1.1rem;
    margin-top: 15px;
}

.upload-section {
    background: white;
    padding: 40px;
    border-radius: 20px;
    box-shadow: 0 10px 25px rgba(0,0,0,0.05);
    display: flex;
    flex-direction: column; 
    align-items: center;
    margin-top: 20px;
    width: 90%; 
    max-width: 900px; /* Increased from 600px to 900px */
}

.custom-file-upload {
    width: 100%; /* Stretches to fill the 900px box */
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 25px; 
    border: 2px dashed #b6f36a;
    border-radius: 12px;
    cursor: pointer;
    background: #fafafa;
    color: #636e72;
    transition: all 0.3s ease;
    min-height: 80px; /* Made it a bit taller too */
    margin-bottom: 20px;
}

.start-btn {
    width: 250px; /* Gives the button a solid, fixed width */
    padding: 15px;
    border-radius: 12px;
    background: #7ed957;
    color: white;
    font-weight: bold;
    border: none;
}
</style>

<h1 class="main-header">Reddit Sentiment Analysis</h1>

<div class="landing-page">
    
    <div class="info-section">
        <h2>Uncover Mental Health Stigma</h2>
        <p>This website utilizes Natural Language Processing (NLP) to analyze Reddit discussions sentiment.</p>
        
        <div class="stats-mini">
            <span>VADER Sentiment Analyzer</span>
            <span style="margin: 0 10px;">|</span>
            <span>NLP Powered</span>
        </div>
    </div>

    <div class="upload-section">
        <label for="file-upload" class="custom-file-upload">
            <span>{fileName}</span>
        </label>

        <input 
            id="file-upload" 
            type="file" 
            accept=".csv" 
            on:change={handleFile} 
            style="display: none;" 
        />
        
        <button class="start-btn" on:click={upload}>
            Start Analysis
        </button>
    </div>
</div>