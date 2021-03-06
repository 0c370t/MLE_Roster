<script>
    import {Button, Tile, Flex} from "svelte-uikit3";
    import {link} from "svelte-routing";
    import {SalaryCapByLeague} from "../../helpers/MLE_META";
    import {sortLeaguesInOrder} from "../../helpers/MLE_META";
    import {getAllPlayers} from "../Players/playersHelpers";
    import {getContext, onMount} from "svelte";
    import Tooltip from "../uikit/Tooltip.svelte";

    export let team;
    const firestore = getContext("firebase").getFirebase().firestore();
    let players = false;
    onMount(async () => {
        players = (await getAllPlayers(firestore)).reduce((o, v) => {
            o[v.PLAYERS.MLEID] = v;
            return o;
        }, {});
    })

    let players_by_league = {};
    $: {
        if (players) {
            players_by_league = {};
            team.players.forEach(playerId => {
                let player = players[playerId.id];
                if (!players_by_league.hasOwnProperty(player.PLAYERS.League))
                    players_by_league[player.PLAYERS.League] = [];
                players_by_league[player.PLAYERS.League].push(player);
            });
            players_by_league = players_by_league;
        }
    }
    let totals = {}
    const getSalaryTotal = (league) => {
        if (!Object.keys(totals).includes(league))
            totals[league] = players_by_league[league]
                .filter(p => !p.PLAYERS.Role.includes("Reserve"))
                .reduce((acc, p) =>
                    acc + parseFloat(p.SALARY.Salary)
                    , 0)
        return totals[league];
    }
    const getSalaryOffset = (league) => {
        if (!Object.keys(totals).includes(league)) getSalaryTotal(league);
        let total = totals[league];
        if (league === "Premier") return "N/A";
        return SalaryCapByLeague[league] - total;
    }
</script>

<h3 class="uk-text-center">Team Roster</h3>

<div class="uk-flex uk-flex-wrap">
    <Tile width="1-1" style="secondary" class="uk-padding-small">
        <h3>Leadership</h3>
        <dl class="uk-description-list">
            <dt>General Manager</dt>
            <dd>
                <a href="/players/{team.leadership.General_Manager.mleid}" class="uk-link-reset" use:link>
                    {team.leadership.General_Manager.name}
                </a>
            </dd>
            {#if team.leadership.Assistant_GM && team.leadership.Assistant_GM.mleid > 0}
                <dt>Assistant General Manager</dt>
                <dd>
                    <a href="/players/{team.leadership.Assistant_GM.mleid}" class="uk-link-reset" use:link>
                        {team.leadership.Assistant_GM.name}
                    </a>
                </dd>
            {/if}
            <dt>Captain(s)</dt>
            {#if Array.isArray(team.leadership.Captain)}
                {#each team.leadership.Captain as cap}
                    {#if cap && cap.mleid > 0}
                        <dd>
                            <a href="/players/{cap.mleid}" class="uk-link-reset" use:link>
                                {cap.name}
                            </a>
                        </dd>
                    {/if}
                {/each}
            {:else if team.leadership.Captain && team.leadership.Captain.mleid > 0}
                <dd>
                    <a href="/players/{team.leadership.Captain.mleid}" class="uk-link-reset" use:link>
                        {team.leadership.Captain.name}
                    </a>
                </dd>
            {/if}
        </dl>
    </Tile>
    {#each Object.keys(players_by_league).sort(sortLeaguesInOrder) as league}
        {#if players_by_league[league].filter(p => p.PLAYERS.Role.length).length > 0}
            <Tile width="1-1" style="primary" class="uk-width-1-2@s uk-width-1-1@l uk-padding-small">
                <h3>
                    {league}<br class="uk-hidden@m"/> ( <Tooltip title="Total Salary">{ getSalaryTotal(league) }</Tooltip> | <Tooltip title="Cap Differential">{ getSalaryOffset(league)}</Tooltip> ) </h3>
                <ul>
                    <li>
                        <h4>Starters</h4>
                    </li>
                    {#each players_by_league[league].filter(p => p.PLAYERS.Role.includes("Player")) as player}
                        <li>
                            <a href="/players/{player.PLAYERS.MLEID}" use:link>
                                <Button style="text">
                                    {player.PLAYERS.Player} ({player.SALARY.Salary})
                                </Button>
                            </a>
                        </li>
                    {/each}
                    <hr/>
                    <li>
                        <h4>Reserve</h4>
                    </li>
                    {#each players_by_league[league].filter(p => p.PLAYERS.Role.includes("Reserve")) as player}
                        <li>
                            <a href="/players/{player.PLAYERS.MLEID}" use:link>
                                <Button style="text">
                                    {player.PLAYERS.Player} ({player.SALARY.Salary})
                                </Button>
                            </a>
                        </li>
                    {/each}

                </ul>
                <hr/>
            </Tile>
        {/if}
    {/each}
</div>

<style lang="scss">
    @media screen and (min-width: 960px) {
        div.uk-flex {
            flex-direction: column;
        }
    }

</style>
