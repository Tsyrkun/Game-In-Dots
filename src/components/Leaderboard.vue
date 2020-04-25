<template>
    <div class="col-sm">
        <h3>Leader Board</h3>
        <table class="table">
            <thead>
              <tr>
                  <th scope="col">User Name</th>
                  <th scope="col">Date and Time</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="user in winners" :key="user.id">
                  <td>{{user.winner}}</td>
                  <td>{{user.date}}</td>
              </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    import axios from 'axios'
    import eventBus from "../eventBus";

    export default {
        name: 'LeaderBoard',
        data: () => ({
            winners: []
        }),
        mounted() {
            this.fetchWinners();
            eventBus.$on('updateLeaderBoard', this.fetchWinners)
        },
        methods: {
            async fetchWinners() {
                try {
                    let res = await axios.get('https://starnavi-frontend-test-task.herokuapp.com/winners');
                    this.winners = res.data;
                } catch (err) {
                    console.log(err);
                }
            }
        }
    }
</script>

<style lang="scss">

</style>
