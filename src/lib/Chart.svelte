<script lang="ts">
    import * as Plot from "@observablehq/plot";

    export let debtArray: any[] = [];
    export let sliderVal: number;
    export let itemWidth: number;

    $: chartArray = setData(debtArray, itemWidth);

    const setData = (debt: any[], width: number): any[] => {
        if (!debt[0]) return [];

        const chartArray = debt.map(({ record_date, tot_pub_debt_out_amt }) => {
            const date = new Date(record_date);
            const debt: number = Number.parseFloat(tot_pub_debt_out_amt);
            const obj = {
                date,
                debt
            };
            return obj;
            }
        );

        const maxVal = chartArray.reduce((prev, current) => (prev && prev.debt > current.debt) ? prev : current).debt;

        const graph = Plot.plot({
            width: width,
            height: 500,
            x: {
                label: "Date",
            },
            y: {
                label: "Debt Amount",
                grid: true,
                base: 100000000,
                tickFormat: d => `${d > chartArray[0].debt ? "+" : ""}${(d / maxVal)}%`
            },
            marks: [
                Plot.line(chartArray, { x: "date", y: "debt" }),
            ],
        });
        const div = document.querySelector("#myplot");
        if(div) {
            div.innerHTML = "";
            div.append(graph);
        }
        // if (div) div.append(graph);

        return chartArray;
    };
</script>

<div id="myplot" />

<style>
    #myplot {
        flex: 5;
    }
</style>
