<script>
    import {Card, Flex, Button} from 'svelte-uikit3';
    import Icon from "../components/uikit/Icon.svelte";
    import Body from "../components/layout/Body.svelte";
    import {link} from 'svelte-routing';

</script>


<Body>
<Card width="1-1" class="uk-margin-medium">
    <h2 slot="header">Info Lookup</h2>
    <Flex direction="column" class="uk-margin-small" alignment="middle">
        <a use:link href="/players" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="user"/>
                By Player
            </Button>
        </a>
        <hr class="uk-width-1-2"/>
        <a use:link href="/teams" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="users"/>
                By Team
            </Button>
        </a>
        <hr class="uk-width-1-2"/>
        <a use:link href="/detail/roster" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="grid"/>
                Compare Rosters
            </Button>
        </a>
    </Flex>
</Card>
<Card width="1-1" class="uk-margin-medium">
    <h2 slot="header">Season Info Lookup</h2>
    <h3 class="uk-text-center">Season 11</h3>
    <hr class="uk-width-3-4 uk-margin-auto"/>
    <Flex direction="column" class="uk-margin-small" alignment="middle">
        <a use:link href="/season/11" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="calendar"/>
                Series Stats By Match
            </Button>
        </a>
        <hr class="uk-width-1-2"/>
        <a use:link href="/detail/teamstats" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="calendar"/>
                Aggregate Stats By Team
            </Button>
        </a>
        <hr class="uk-width-1-2"/>
        <a use:link href="/detail/playerstats" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="calendar"/>
                Aggregate Stats By Player
            </Button>
        </a>
        <hr class="uk-width-1-2"/>
        <a use:link href="/season/11/standings" class="uk-width-3-4 uk-width-1-3@m">
            <Button width="1-1">
                <Icon icon="list"/>
                Team Standings
            </Button>
        </a>
    </Flex>
</Card>
</Body>

<style>
    main {
        margin: 4em auto;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
</style>