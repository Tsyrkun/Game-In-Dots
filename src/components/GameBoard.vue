<template>
    <div class="col-sm">
        <form class="form-inline" @submit.prevent>
            <select :disabled="playing" v-model="selectedMode" class="custom-select" @change="setModeData">
                <option disabled selected value="empty">Pick game mode</option>
                <option v-for="(value, key) in settings" :value="key">{{key}}</option>
            </select>

            <input :disabled="playing" v-model="username" type="text" class="form-control ml-3"
                   placeholder="Enter your name">

            <button id="play" @click="start" :disabled="!canStart || playing" class="btn btn-primary ml-3">Play</button>
        </form>

        <div class="font-weight-bold lead text-center my-4" id="message"></div>

        <div class="grid" :style="gridStyles">
            <button
                    :style="{backgroundColor: cell.status === 'move' ? 'blue' : cell.status === 'comp' ? 'red' : cell.status === 'user' ? 'green' : ''}"
                    @click="userClick(index)" :disabled="!playing"
                    v-for="(cell, index) in cells"
                    :key="index"
            />
        </div>
    </div>
</template>

<script>
    import axios from 'axios'
    import eventBus from "../eventBus";

    export default {
        name: "GameBoard",
        data: () => ({
            settings: {},
            username: '',
            selectedMode: 'empty',
            selectedModeData: {},
            emptyIndexes: [],
            cells: [],
            currentCellIndex: null,
            maxIndexLength: 0,
            timer: null,
            playing: false,
            result: {
                id: 0,
                winner: '',
                date: ''
            }
        }),
        mounted() {
            this.fetchSettings()
        },
        watch: {
            selectedModeData() {
                const count = Math.pow(this.selectedModeData.field, 2);
                this.cells = new Array(count).fill({}).map(c => ({
                    status: 'empty'
                }))
            },
            computerScore(score) {
                const won = this.cells.length/2;
                if (score >= won) {
                    this.end();
                    this.result.winner = 'Computer';
                    this.result.date = this.getCurrentDate();
                }
            },
            userScore(score) {
                const won = this.cells.length/2;
                if (score >= won) {
                    this.end();
                    this.result.winner = this.username;
                    this.result.date = this.getCurrentDate();
                }
            },
            playing(status) {
                if (!status) {
                    document.querySelector('#play').innerHTML = 'Play again';
                    document.querySelector('#message').innerHTML = this.result.winner + ' won';
                }
            }
        },
        computed: {
            isModeSelected() {
                return this.selectedMode !== 'empty'
            },

            hasUsername() {
                return !!this.username || !!this.username.length
            },

            canStart() {
                return this.hasUsername && this.isModeSelected
            },

            gridStyles() {
                return {
                    gridTemplateColumns: `repeat(${this.selectedModeData.field}, 50px)`,
                    gridTemplateRows: `repeat(${this.selectedModeData.field}, 50px)`
                }
            },

            computerScore() {
                return this.cells.filter(c => c.status === 'comp').length
            },

            userScore() {
                return this.cells.filter(c => c.status === 'user').length
            }
        },
        methods: {
            async fetchSettings() {
                try {
                    let res = await axios.get('https://starnavi-frontend-test-task.herokuapp.com/game-settings');
                    this.settings = res.data;
                } catch (err) {
                    console.log(err);
                }
            },

            setModeData(e) {
                this.selectedModeData = this.settings[e.target.value]
            },

            start() {
                this.clearMessage();
                this.playing = true;
                this.maxIndexLength = Math.pow(this.selectedModeData.field, 2);
                this.emptyIndexes = [...Array(this.maxIndexLength).keys()];
                this.timer = setInterval(this.loop, this.selectedModeData.delay)
            },

            end() {
                this.playing = false;
                clearInterval(this.timer);
                this.clearBoard();
                /*this.postResult();*/
            },

            userClick(index) {
                if (this.cells[index].status === 'move') {
                    this.cells[index].status = 'user'
                }
            },

            loop() {
                if (this.currentCellIndex !== null) {
                    if (this.cells[this.currentCellIndex].status === 'move') {
                        this.cells[this.currentCellIndex].status = 'comp'
                    }
                }
                const index = this.getRandomCell(this.emptyIndexes);

                this.emptyIndexes.splice(this.emptyIndexes.indexOf(index), 1);
                this.currentCellIndex = index;
                this.cells[index].status = 'move'
            },

            getRandomCell(array) {
                return array[Math.floor(Math.random() * (array.length))];
            },

            clearBoard() {
                this.result = {
                    id: 0,
                    winner: '',
                    date: ''
                };
                this.currentCellIndex = null;

                const count = Math.pow(this.selectedModeData.field, 2);

                this.cells = new Array(count).fill({}).map(c => ({
                    status: 'empty'
                }))
            },

            clearMessage() {
                document.querySelector('#message').innerHTML = '';
            },

            getCurrentDate() {
                let date = new Date();

                let options = {
                    hour: 'numeric',
                    minute: 'numeric'
                };

                let options2 = {
                    year: 'numeric',
                    month: 'long'
                };

                return date.toLocaleString("ru", options) + '; ' + date.getDate() + ' ' + date.toLocaleDateString('en-US', options2)
            },

            async postResult() {
                try {
                    await axios.post('https://starnavi-frontend-test-task.herokuapp.com/winners', this.result);
                    eventBus.$emit('updateLeaderBoard')
                } catch (err) {
                    console.log(err);
                }
            }
        }
    }
</script>

<style lang="scss">
    .grid {
        display: grid;
        grid-template-columns: repeat(5, 50px);
        grid-template-rows: repeat(5, 50px);
        grid-gap: 5px;
        gap: 5px;
        justify-content: center;
    }
    .form-inline {
        margin: 0 auto;
        width: fit-content;
    }
</style>