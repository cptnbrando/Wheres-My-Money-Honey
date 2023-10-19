<script lang="ts">
    import { onMount } from "svelte";

    let today: number;
    let thisYear: number;

    let allYears: number[] = [];

    let debtArray: any[] = [];

    let selectedYear: number;
    let selectedYearDropdown: HTMLSelectElement;

    onMount(() => {
        const now = new Date();
        thisYear = now.getFullYear();
        today = Date.now();

        getDebt("2023-01-01");
        fillYears();
    });

    /**
     * This fetches the national debt data from the api, with a given date interval
     * If lt is omitted, this will fetch 500 records after the given gt date
     * @param gt greater than this date in "year-month-day" format
     * @param lt less than this date in "year-month-day" format
     */
    const getDebt = (gt: string, lt?: string): void => {
        const lessThan = lt ? `,record_date:lt:${lt}` : '';
        fetch(`https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?filter=record_date:gt:${gt}${lessThan}&page[size]=500`)
        .then((response) => response.json())
        .then((data) => {
            console.log(data.data);
        })
        .catch((error) => {
            console.log(error);
            return [];
        });
    }

    /**
     * Debt to the Penny API keeps track of debt from 1993 - Now
     * This function will fill the allYears array with all those years, starting with the current year
     */
    const fillYears = (): void => {
        for(let i = thisYear - 1; i > 1992; i--) {
            allYears.push(i);
        }
    }

    const yearChange = (): void => {
        console.log(selectedYear);
    }

    /**
     * Returns a given date with the format "[year]-[month]-[day]"
     * Adds leading 0s if necessary
     * @param date - a Date object
     */
    const formatDate = (date: Date): string => {
        const day = (date.getDay() < 10) ? `0${date.getDay()}` : date.getDay();
        const month = (date.getMonth() < 10) ? `0${date.getMonth()}` : date.getMonth();
        return `${date.getFullYear()}-${month}-${day}`;
    }
</script>

<main>
    <nav>
        <h4>Today</h4>
        <h2>Where's My Money?</h2>
        <span class="inline-flex">
            <div>-</div>
            <select name="year" id="yearSelect" bind:value={selectedYear} bind:this={selectedYearDropdown} on:change={yearChange}>
                <option value={thisYear} selected>{thisYear}</option>
                {#each allYears as year}
                    <option value={year}>{year}</option>
                {/each}
            </select>
            <div>+</div>
        </span>
    </nav>

    <article>
        <div>
            <input type="range" min="1" max={debtArray.length} value="50" class="slider" id="myRange">
        </div>
    </article>

</main>

<style>
    main {
        display: flex;
        flex-direction: column;
    }

    nav {
        flex: 0.2;
        display: flex;
        justify-content: space-between;
        align-items: center;
        border-bottom: solid;
    }

    .inline-flex {
        display: flex;
        flex-direction: row;
        align-items: center;
    }

    .inline-flex select {
        -webkit-appearance: none;
        -moz-appearance: none;
        text-indent: 1px;
        text-overflow: '';
        padding: 5px 5px 4px 4px;
    }

    .inline-flex * {
        margin: 10px;
    }
</style>
