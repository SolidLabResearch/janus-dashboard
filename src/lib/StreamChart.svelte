<script lang="ts">
    import * as echarts from "echarts";
    import { onDestroy, onMount } from "svelte";

    export let isRunning: boolean = false;
    export let query: string = "";

    let chartContainer: HTMLDivElement;
    let chartInstance: echarts.ECharts;
    let intervalId: number;
    let ws: WebSocket | null = null;

    // Mutable buffer in the scope of the script.
    const dataBuffer: {
        time: number;
        live: number;
        hist: number;
    }[] = [];

    onMount(() => {
        chartInstance = echarts.init(chartContainer);

        // Initial Chart Configuration
        chartInstance.setOption({
            backgroundColor: "transparent",
            animation: false,
            tooltip: {
                trigger: "axis",
                backgroundColor: "rgba(255, 255, 255, 0.95)",
                borderColor: "#eee",
                textStyle: {
                    color: "#333",
                },
                extraCssText: "box-shadow: 0 2px 8px rgba(0,0,0,0.1); border-radius: 8px;",
            },
            legend: {
                show: true,
                top: "10px",
                right: "20px",
                textStyle: {
                    color: "#333",
                    fontWeight: "bold"
                },
                itemGap: 20
            },
            grid: {
                left: "20px",
                right: "20px",
                bottom: "20px",
                top: "60px", /* Increased top to make room for legend */
                containLabel: true,
            },
            xAxis: {
                type: "time",
                splitLine: { show: false },
                axisLabel: { color: "#666", fontWeight: "bold" },
                axisLine: { lineStyle: { color: "#ccc" } },
            },
            yAxis: {
                type: "value",
                min: 0,
                max: 100,
                splitLine: { lineStyle: { color: "#eee" } },
                axisLabel: { color: "#666" },
            },
            series: [
                {
                    name: "Live Stream",
                    type: "line",
                    showSymbol: false,
                    smooth: true,
                    // STYLE: Blue, Opaque, Thicker
                    lineStyle: {
                        color: "#2196f3",
                        width: 3,
                        shadowColor: "rgba(33, 150, 243, 0.3)",
                        shadowBlur: 10,
                    },
                    areaStyle: {
                        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                            { offset: 0, color: "rgba(33, 150, 243, 0.2)" },
                            { offset: 1, color: "rgba(33, 150, 243, 0)" },
                        ]),
                    },
                    data: [],
                },
                {
                    name: "Historical Baseline",
                    type: "line",
                    showSymbol: false,
                    smooth: true,
                    // STYLE: Purple, Dashed
                    lineStyle: {
                        type: "dashed",
                        color: "#9c27b0",
                        opacity: 0.6,
                        width: 2,
                    },
                    data: [],
                },
            ],
        });

        const handleResize = () => chartInstance.resize();
        window.addEventListener("resize", handleResize);

        // Force a resize after a short delay to ensure container is ready
        setTimeout(() => {
            chartInstance.resize();
        }, 100);

        return () => {
            window.removeEventListener("resize", handleResize);
            chartInstance.dispose();
        };
    });

    $: if (isRunning) {
        startLoop();
        connectBackend();
    } else {
        stopLoop();
        disconnectBackend();
    }

    function connectBackend() {
        // TODO: Replace with your actual backend WebSocket URL
        const wsUrl = "ws://localhost:8080/query";

        try {
            ws = new WebSocket(wsUrl);

            ws.onopen = () => {
                console.log("WebSocket connected");
                // Send the query to the backend
                if (ws && query) {
                    ws.send(query);
                    console.log("Sent query to backend:", query);
                }
            };

            ws.onmessage = (event) => {
                console.log("Received from backend:", event.data);
                // TODO: Parse and use real data from backend
                // For now, we continue using mock data
            };

            ws.onerror = (error) => {
                console.error("WebSocket error:", error);
            };

            ws.onclose = () => {
                console.log("WebSocket disconnected");
            };
        } catch (error) {
            console.error("Failed to connect to backend:", error);
        }
    }

    function disconnectBackend() {
        if (ws) {
            ws.close();
            ws = null;
        }
    }

    function startLoop() {
        if (intervalId) return;

        intervalId = window.setInterval(() => {
            const now = new Date().getTime();

            /// Change in future with websocket based results.
            const histVal = 50 + Math.sin(now / 1000) * 30;
            const liveVal =
                50 + Math.sin(now / 1000 + 0.2) * 30 + (Math.random() * 10 - 5);

            dataBuffer.push({
                time: now,
                live: liveVal,
                hist: histVal,
            });

            if (dataBuffer.length > 500) {
                dataBuffer.shift();
            }

            // console.log("Updating chart, buffer size:", dataBuffer.length); 
            // Uncommenting the above log might flood console, but good for debugging once.
            
            if (chartInstance) {
                chartInstance.setOption({
                    series: [
                        { data: dataBuffer.map((d) => [d.time, d.live]) },
                        { data: dataBuffer.map((d) => [d.time, d.hist]) },
                    ],
                });
            } else {
                console.warn("Chart instance not ready yet");
            }
        }, 16);
        console.log("Started data loop");
    }

    function stopLoop() {
        clearInterval(intervalId);
        intervalId = 0;
    }

    onDestroy(() => {
        stopLoop();
        disconnectBackend();
    });
</script>

<div class="chart-wrapper" bind:this={chartContainer}></div>

<style>
    .chart-wrapper {
        flex: 1;
        width: 100%;
        height: 600px; /* Force explicit height */
        min-height: 400px;
        min-width: 500px; /* Prevent squashing */
        border-radius: 12px;
        background: #ffffff;
        border: 1px solid #eee;
        box-shadow: 0 4px 20px rgba(0,0,0,0.02);
    }
</style>
