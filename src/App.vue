<template>
    <div class="container-fluid p-5">
        <div class="row">
            <GameBoard/>
            <LeaderBoard/>
        </div>
    </div>
</template>

<script>
    import LeaderBoard from "./components/Leaderboard";
    import GameBoard from "./components/GameBoard";

    export default {
        name: 'App',
        components: {
            GameBoard,
            LeaderBoard
        }
    }
</script>

<style lang="scss">

</style>
