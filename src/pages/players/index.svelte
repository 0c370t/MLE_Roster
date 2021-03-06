<script>
    import {fade} from "svelte/transition";
    import {Tile, Flex, uk_width, Drop, Button, ButtonGroup} from "svelte-uikit3";
    import Icon from "../../components/uikit/Icon.svelte";
    import Body from "../../components/layout/Body.svelte";
    import {collectionStore} from "sveltefire";
    import {getContext, onDestroy, onMount} from "svelte";
    import {link} from "svelte-routing";
    import Spinner from "../../components/uikit/Spinner.svelte";
    import FilterPanel from "../../components/Players/FilterPanel.svelte";
    import {filterPlayers, getAllPlayers, getMaxCurrentSalary} from "../../components/Players/playersHelpers";
    import PageControl from "../../components/PageControl.svelte";
    import TeamLogo from "../../components/TeamLogo.svelte";
    import {allPlayersStoreFactory} from "../../helpers/firebase/FirestoreCacheStoreFactory";

    $: [players, pages] = filterPlayers(_players, order, asc, query, pageSize, pages, page, minSalary, maxSalary, team, minMMR, maxMMR)
    $: if (order || asc || query || pageSize || minSalary || maxSalary || team || minMMR || maxMMR) page = 0;
    const setOrder = newOrder => {
        if (newOrder === order) {
            asc = !asc;
        } else {
            order = newOrder;
            asc = false;
        }
        page = 0;
    };
    const notificationStore = getContext("notificationStore");

    let _playersPromise = allPlayersStoreFactory();
    let _players = [];
    let players = [];
    let teams = [], team = "";
    let loading = true;


    // Salaries
    let _minSalary = 0;
    let _maxSalary = 22;

    let minSalary = _minSalary.toFixed(1);
    let maxSalary = _maxSalary.toFixed(1);

    // MMRs
    let _minMMR = 0;
    let _maxMMR = 0;

    let minMMR = _minMMR;
    let maxMMR = _minMMR;

    // Sorting / Filtering
    let query = "";
    let order = "name";
    let asc = false;
    // Pagination
    let page = 0;
    let pageSize = 20;
    let pages;

    const unsub = _playersPromise.subscribe(v => {
        if(v.hasOwnProperty("loading")) return;
        players = v;
        _players = v;
        teams = [...new Set(v.map(p => p.PLAYERS.Team))].sort();
        loading = false;
        const all_salaries = [...v.map(p => p.SALARY.Salary)]
        _minSalary = Math.min(...all_salaries);
        _maxSalary = Math.max(...all_salaries);
        const all_mmrs = [...v.map(p => getMaxCurrentSalary(p))].filter(v => v > Number.MIN_SAFE_INTEGER);
        _minMMR = Math.min(...all_mmrs);
        _maxMMR = Math.max(...all_mmrs);
        minMMR = _minMMR;
        maxMMR = _maxMMR;
    });

    onDestroy(()=>unsub)

    let currentMMRHasBeenNotified = false;
    function currentMMRNotification(){
        if(!currentMMRHasBeenNotified){
            notificationStore.set({
                message: "Current MMR may not show all players, and may not be accurate due to issues with the Psyonix API.",
                status: "warning",
                timeout: 10000,
            })
        }
        currentMMRHasBeenNotified = true;
        return true;
    }

</script>

<Body>

<div class="uk-inline uk-width-3-4 uk-margin-small uk-margin-auto uk-width-expand">
        <span>
            <Icon _class="uk-form-icon uk-form-icon-flip" icon="search"/>
        </span>
    <input class="uk-input" bind:value={query} placeholder="Search...">
</div>
<div class="uk-width-1-1 uk-width-3-4@m">
    <div class="uk-flex uk-flex-column uk-flex-middle uk-margin-small">
        <span>Order By: </span>
        <ButtonGroup>
            <Button style={order === "name" ? "primary" : "default"} on:click={()=>setOrder("name")} class="uk-text-nowrap">
                Name
                {#if order === "name"}
                    <Icon icon={asc ? "chevron-down" : "chevron-up"}/>
                {/if}
            </Button>
            <Button style={order === "salary" ? "primary" : "default"} on:click={()=>setOrder("salary")} class="uk-text-nowrap">
                Salary
                {#if order === "salary"}
                    <Icon icon={asc ? "chevron-down" : "chevron-up"}/>
                {/if}
            </Button>
            <Button style={order === "currentmmr" ? "primary" : "default"} on:click={()=>currentMMRNotification() && setOrder("currentmmr")} class="uk-text-nowrap">
                Current MMR
                {#if order === "currentmmr"}
                    <Icon icon={asc ? "chevron-down" : "chevron-up"}/>
                {/if}
            </Button>
        </ButtonGroup>
    </div>
    <FilterPanel {_maxSalary} {_minSalary} {_minMMR} {_maxMMR} {teams} bind:team bind:maxSalary bind:minSalary bind:minMMR bind:maxMMR/>
</div>
<PageControl {pages} bind:page/>
<hr class="uk-width-1-1 uk-margin-small"/>
<div class="uk-width-1-1">
    <Spinner show={loading}/>
    {#if players}
        {#each players as player, i (player.PLAYERS.MLEID)}
            <a use:link href="/players/{player.PLAYERS.MLEID}" use:uk_width={"1-1"} class="uk-link-reset" in:fade>
                <div class="uk-width-1-1 uk-margin-small uk-padding-small uk-box-shadow-hover-medium uk-tile uk-tile-primary uk-preserve-color {player.PLAYERS.Team.toLowerCase()}">
                    <Flex justification="between" alignment="middle">
                        <Flex alignment="middle">
                            <TeamLogo/>
                            <h3 class="uk-margin-remove player-name">
                                {player.PLAYERS.Player}
                            </h3>
                        </Flex>
                        <Flex alignment="middle">
                            <span class="uk-margin-remove uk-visible@m">{player.PLAYERS.Team}</span>
                            <hr class="uk-divider-vertical uk-margin-small-right uk-margin-small-left uk-visible@m" style="height:2em;">
                            <span class="uk-margin-remove uk-visible@m">{player.PLAYERS.League}</span>
                            <hr class="uk-divider-vertical uk-margin-small-right uk-margin-small-left uk-visible@m" style="height:2em;">
                            <span class="uk-margin-remove">{player.SALARY.Salary}</span>
                        </Flex>
                    </Flex>
                </div>
            </a>
        {/each}
    {/if}
</div>
<hr class="uk-width-1-1 uk-margin-small"/>
<PageControl {pages} bind:page/>

</Body>

<style lang="scss">
    h3.player-name{
        font-size:1.1em;
        color:inherit;
    }
    div.uk-tile {
        background-color: var(--background-color);
        color: var(--color);
    }
    @media screen and (min-width: 960px){
        .player-name{
            font-size:1.5em;
        }
    }
</style>