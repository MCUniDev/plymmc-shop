<script lang="ts">
    import { Line } from 'svelte-chartjs';
    import type { DataPoint } from './types';

    import {
        Chart as ChartJS,
        Tooltip,
        LineElement,
        LinearScale,
        PointElement,
        CategoryScale,
        Legend,
        type ChartData,
    } from 'chart.js';

    export let point: DataPoint;

    ChartJS.register(
        Tooltip,
        LineElement,
        LinearScale,
        PointElement,
        CategoryScale,
        Legend,
    );

    let chartData: ChartData<'line'>;
    $: chartData = {
        labels: Array.from({ length: point[1].length }, (_, i) => i + 1),
        datasets: [
            {
                borderColor: '#67c157',
                borderCapStyle: 'butt',
                borderJoinStyle: 'round',
                pointBorderColor: '#67c157',
                pointBackgroundColor: '#67c157',
                pointHoverBackgroundColor: 'rgb(0, 0, 0)',
                pointHoverBorderColor: 'rgba(220, 220, 220, 1)',
                label: 'Buy Price',
                data: point[1],
            },
            {
                borderCapStyle: 'butt',
                borderJoinStyle: 'round',
                borderColor: '#f26c4b',
                pointBorderColor: '#f26c4b',
                pointBackgroundColor: '#f26c4b',
                pointHoverBackgroundColor: 'rgb(0, 0, 0)',
                pointHoverBorderColor: 'rgba(220, 220, 220, 1)',
                label: 'Sell Price',
                data: point[1].map((d) => d * 0.25),
            },
        ],
    };
</script>

<Line
    data={chartData}
    options={{
        responsive: true,
        scales: {
            x: { title: { display: true, text: 'Time (15 Minute Intervals)' } },
            y: { title: { display: true, text: 'Price per item' } },
        },
    }} />
