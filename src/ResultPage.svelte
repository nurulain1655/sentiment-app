<script>
    import { onMount } from 'svelte';
    import Chart from "chart.js/auto";
    import ChartDataLabels from "chartjs-plugin-datalabels";
    import WordCloud from "wordcloud";  
    import snarkdown from 'snarkdown';

    Chart.register(ChartDataLabels);

    export let result;
    export let aiInsight;    
    export let loading;      
    export let getLlamaInsight;
    export let goBack; // Added to match your App.svelte back action

    // --- Chart Tracking Instances ---
    let chart;    // Pie Chart Instance
    let barChart; // NEW: Added missing barChart instance declaration!

    $: if (result) {
        setTimeout(() => {
            drawChart();
            drawWordCloud();
            drawBarChart();
        }, 0);
    }

    function drawBarChart() {
        const ctx = document.getElementById("barChartCanvas");
        if (!ctx || !result || !result.timeline) return;

        if (barChart) barChart.destroy();

        const labels = result.timeline.map(item => item.month);
        const positiveData = result.timeline.map(item => item.positive);
        const negativeData = result.timeline.map(item => item.negative);
        const neutralData = result.timeline.map(item => item.neutral);

        barChart = new Chart(ctx, {
            type: "bar",
            data: {
                labels: labels,
                datasets: [
                    {
                        label: "Positive",
                        data: positiveData,
                        backgroundColor: "#4caf50"
                    },
                    {
                        label: "Negative",
                        data: negativeData,
                        backgroundColor: "#f44336"
                    },
                    {
                        label: "Neutral",
                        data: neutralData,
                        backgroundColor: "#ffc107"
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    x: { 
                        stacked: false, 
                        title: { display: true, text: "Timeline (Months)" }
                    },
                    y: { 
                        stacked: false,
                        title: { display: true, text: "Number of Posts" },
                        beginAtZero: true 
                    }
                },
                plugins: {
                    datalabels: { display: false } 
                }
            }
        });
    }

    function drawChart() {
        const ctx = document.getElementById("pieChart");
        if (!ctx) return;

        if (chart) chart.destroy();

        chart = new Chart(ctx, {
            type: "pie",
            data: {
                labels: ["Positive", "Negative", "Neutral"],
                datasets: [
                    {
                        data: [
                            result.positive,
                            result.negative,
                            result.neutral
                        ],
                        backgroundColor: [
                            "#4caf50",
                            "#f44336",
                            "#ffc107"
                        ]
                    }
                ]
            },
            options: {
                plugins: {
                    datalabels: {
                        color: "white",
                        font: { weight: "bold", size: 16 },
                        formatter: (value, ctx) => {
                            const sum = ctx.chart.data.datasets[0].data.reduce((a,b)=>a+b,0);
                            return sum > 0 ? ((value/sum)*100).toFixed(1)+"%" : "0%";
                        }
                    },
                    legend: {
                        position: "bottom"
                    }
                }
            }
        });
    }

    function drawWordCloud() {
        const canvas = document.getElementById("wordCloudCanvas");
        if (!canvas || !result.word_freq) return;

        const words = Object.entries(result.word_freq);
        const maxFreq = Math.max(...words.map(w => w[1]));

        WordCloud(canvas, {
            list: words,
            gridSize: 8,
            weightFactor: (size) => (size / maxFreq) * 80, 
            fontFamily: 'Times, serif',
            color: 'random-dark',
            rotateRatio: 0.5,
            backgroundColor: '#ffffff'
        });
    }

    onMount(() => {
        if (result && result.text_content) {
            getLlamaInsight(result.text_content);
        }
    });
</script>

<style>
:global(body) {
    background-color: #f0f4f8; 
    font-family: 'Inter', sans-serif;
    margin: 0;
}

h1 {
    text-align: center;
    background: #7ed957;
    color: white;
    padding: 20px;
    margin: 0;
    font-weight: 300;
    letter-spacing: 1px;
}

h2, h3 {
    margin-top: 0%;
}

.container {
    display: grid;
    grid-template-columns: 1fr 1fr; 
    gap: 25px;
    padding: 25px;
    max-width: 1400px;
    margin: 0 auto;
}

.top-row {
    grid-column: 1 / -1; 
    display: flex;
    align-items: center; 
    gap: 40px;
    background: white;
    border-radius: 12px;
    padding: 30px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #e1e8ed;
}

.overview-text {
    flex: 1; 
    min-width: 250px;
}

.pie-container {
    flex: 2; 
    display: flex;
    justify-content: center;
}

#pieChart {
    max-height: 450px !important; 
    width: 100% !important;
}

.bottom-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 25px;
    grid-column: 1 / -1;
}

.chartBox {
    background: white;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #e1e8ed;
}

.ai-report {
    grid-column: 2; 
    background: white;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #e1e8ed;
    height: 100%;   
    display: flex;
    flex-direction: column;
}

.timeline-chart-box {
    grid-column: 1 / -1; 
    background: white;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #e1e8ed;
    margin-top: 10px;
}

/* FIXED: Closed the dangling property brackets layout */
.chart-container {
    position: relative;
    height: 350px; 
    width: 100%;
}

.timeline-box {
    grid-column: 1 / -1; 
    background: white;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #e1e8ed;
    margin-top: 25px;
}

.timeline-wrapper {
    display: flex;
    overflow-x: auto; 
    gap: 20px;
    padding: 15px 5px;
}

.timeline-node {
    min-width: 220px;
    background: #f8f9fa;
    border-left: 4px solid #7ed957; 
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.02);
}

.node-date {
    font-weight: bold;
    color: #2d3436;
    font-size: 1.1rem;
    margin-bottom: 8px;
}

.topic-tag {
    margin: 5px 0;
    font-size: 0.95rem;
    color: #2d3436;
}

.mini-bar {
    font-size: 0.85rem;
    margin-top: 10px;
}

.nav-btn {
    display: block;
    margin: 20px auto;
    padding: 12px 30px;
    background: #7ed957;
    color: white;
    border: none;
    border-radius: 8px;
    font-weight: bold;
    cursor: pointer;
}
</style>

<h1>Sentiment Analysis Result</h1>

<div class="container">
    {#if result}
        <div class="top-row">
            <div class="overview-text">
                <h3 style="color: #636e72;">Analysis Overview</h3>
                <p style="font-size: 1.2rem;"><strong>Total Data:</strong> {result.total}</p>
                <hr style="border: 0; border-top: 1px solid #eee; margin: 15px 0;">
                <p style="color: #4caf50; font-weight: bold;">✔ Positive: {result.positive}</p>
                <p style="color: #f44336; font-weight: bold;">✖ Negative: {result.negative}</p>
                <p style="color: #ffc107; font-weight: bold;">● Neutral: {result.neutral}</p>
            </div>

            <div class="pie-container">
                <canvas id="pieChart"></canvas>
            </div>
        </div>

        <div class="bottom-container">
            <div class="chartBox">
                <h3>Word Cloud</h3>
                <canvas id="wordCloudCanvas" width="600" height="400"></canvas>
            </div>

            <section class="ai-report">
                <h3>📊 Automated Discourse Insight</h3>
                <div class="insight-box">
                    {#if loading}
                        <p>Llama 3 is synthesizing emotional trends...</p>
                    {:else if aiInsight}
                        <div>{@html snarkdown(aiInsight)}</div>
                    {:else}
                        <p>Waiting for data analysis...</p>
                    {/if}
                </div>
            </section>
        </div>

        <div class="timeline-chart-box">
            <h3>📈 Sentiment Distribution Over Time</h3>
            <div class="chart-container">
                <canvas id="barChartCanvas"></canvas>
            </div>
        </div>
        {#if goBack}
            <button class="nav-btn" on:click={goBack}>Analyze Another Dataset</button>
        {/if}
    {:else}
        <p style="grid-column: 1/-1; text-align: center;">Loading data visualization layers...</p>
    {/if}
</div>