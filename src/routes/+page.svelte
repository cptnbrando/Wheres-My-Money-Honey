<script lang="ts">
    import { onMount } from "svelte";
    import Chart from "../lib/Chart.svelte";

    let today: string;
    let thisYear: number;

    let allYears: number[] = [];

    let debtArray: any[] = [];

    let selectedYear: number;
    let selectedYearDropdown: HTMLSelectElement;

    let sliderVal: number = 0;

    let innerWidth: number = 0;

    $: itemWidth = 85 * innerWidth / 100;

    let dataDictionary: any = {};

    onMount(async () => {
        // itemWidth = 85 * window.innerWidth / 100;
        const now = new Date();
        thisYear = now.getFullYear();
        selectedYear = thisYear;
        today = `${formatDate(now)}`;

        await getDebt("2023-01-01");
        fillYears();
    });

    /**
     * This fetches the national debt data from the api, with a given date interval
     * If lt is omitted, this will fetch 500 records after the given gt date
     * It will store the data in dataDictionary and use the stored data first
     * 
     * This should be adjusted so we get all records from 1993 at load,
     * parse them into a dictionary by the year,
     * then just play with the dictionary, because we don't wanna overload the government lol
     * they work so hard for us! bad for business, let's help them out... eventually lol
     * @param gt greater than this date in "year-month-day" format
     * @param lt less than this date in "year-month-day" format
     */
    const getDebt = async (gt: string, lt?: string): Promise<void> => {
        if(dataDictionary[gt]) {
            debtArray = dataDictionary[gt];
            sliderVal = debtArray.length;
            return;
        }
        const lessThan = lt ? `,record_date:lt:${lt}` : '';
        await fetch(`https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?filter=record_date:gt:${gt}${lessThan}&page[size]=500`)
        .then((response) => response.json())
        .then((data) => {
            console.log("fetched data");
            console.log(data.data);
            debtArray = data.data;
            sliderVal = debtArray.length;
            dataDictionary[gt] = data.data;
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
        allYears = [];
        for(let i = thisYear - 1; i > 1992; i--) {
            allYears.push(i);
        }
    }

    /**
     * Callback for dropdown year menu
     * Gets the new debt array for the selected year, and sets the slider to the beginning
     */
    const yearChange = async (): Promise<void> => {
        await getDebt(`${selectedYear}-01-01`, `${selectedYear + 1}-01-01`);
        sliderVal = 1;
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

    /**
     * Returns a string from a Date object with the format "[Month] [Day], [Year]"
     * Presented above the slider
     * @param date - a Date object
     */
    const legibleDate = (myDate: string): string => {
        const date: Date = new Date(myDate);
        const month = date.toLocaleString('default', { month: 'long' });
        return `${month} ${date.getDate()}, ${date.getFullYear()}`;
    }

    /**
     * Adds commas to a given money string
     * @param money a big number of dolla dolla bills
     */
    const formatMoney = (money: string): string => {
        return money.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }

    /**
     * Increases or decreases the selectedYear by one
     * Returns early if increasing and selectedYear is the current year, or if decreasing and it is 1993 (the lowest date)
     * @param up true to increase, false to decrease
     */
    const changeYear = async (up: boolean): Promise<void> => {
        if(up) {
            if(selectedYear === thisYear) return;
            selectedYear++;
        }
        else {
            if(selectedYear === 1993) return;
            selectedYear--;
        }

        await yearChange();
    }

    $: selectedDate = (debtArray[sliderVal - 1]) ? debtArray[sliderVal - 1].record_date : today;
    $: money = (debtArray[sliderVal - 1]) ? formatMoney(debtArray[sliderVal - 1].tot_pub_debt_out_amt) : "Money";
</script>

<svelte:window
    bind:innerWidth />
<main>
    <nav>
        <div id="clock">
            <h4>Today</h4>

        </div>
        <h2 id="title">Where's My Money?</h2>
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span class="inline-flex" id="years">
            <div class="clickable" on:click={() => { changeYear(false) }}>-</div>
            <select name="year" id="yearSelect" bind:value={selectedYear} bind:this={selectedYearDropdown} on:change={yearChange}>
                <option value={thisYear} selected>{thisYear}</option>
                {#each allYears as year}
                    <option value={year}>{year}</option>
                {/each}
            </select>
            <div class="clickable" on:click={() => { changeYear(true) }}>+</div>
        </span>
    </nav>

    <article class="col-flex">
        <div id="slider" class="col-flex">
            <span>{legibleDate(selectedDate)}</span>
            <input type="range" min="1" max={debtArray.length} bind:value={sliderVal} class="slider" id="myRange">
        </div>
        <div id="money">
            <h1>${money}</h1>
        </div>
        <Chart {debtArray} {sliderVal} {itemWidth} />
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
        border-bottom: solid;
    }

    nav > * {
        flex: 1;
        display: flex;
        place-content: center;
    }

    #title {
        flex: 5;
    }

    #clock {
        place-content: flex-start;
    }

    #years {
        place-content: flex-end;
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

    article {
        height: 85vh;
        margin-top: 3vh;
        justify-content: space-between;
        align-items: center;
    }

    .col-flex {
        display: flex;
        flex-direction: column;
    }

    #slider {
        align-items: center;
        justify-content: center;
    }

    #slider, #money {
        flex: 1;
    }

    input {
        width: 85vw;
    }

    .clickable {
        cursor: pointer;
        -webkit-user-select: none; /* Safari */
        -ms-user-select: none; /* IE 10 and IE 11 */
        user-select: none; /* Standard syntax */
    }

    @media (max-width: 991.98px) {
        .clickable {
            display: none;
        }
    }
</style>
