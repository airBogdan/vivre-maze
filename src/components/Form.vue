<template>
    <div class="form">
        <form @submit.prevent="createMaze">
            <div>
                <label for="width">width</label>
                <input id="width" name="width" v-model="width" type="text">
            </div>

            <div>
                <label for="height">height</label>
                <input id="height" name="height" v-model="height" type="text">
            </div>

            <div>
                <label for="start">start*</label>
                <input id="start" name="start" v-model="startPoint" type="text">
            </div>

            <div>
                <label for="end">end*</label>
                <input id="end" name="end" v-model="endPoint" type="text">
            </div>

            <div>
                <label for="brickDensity">brickDensity</label>
                <input id="brickDensity" name="brickDensity" v-model="brickDensity" type="text">
            </div>

            <button type="submit">Create maze</button>

        </form>

            <button @click="regenerateMaze">Generate another maze</button>
            <p>* start si end sa inceapa de la 1 si sa fie separate cu virgula</p>
            <p>Start = verde, end = rosu, drumul = albastru</p>

        <h2>{{ solution}}</h2>
    </div>
</template>

<script>
    export default {
        data: () => ({
            width: 6,
            height: 6,
            startPoint: '1,3',
            endPoint: '4,4',
            brickDensity: 34
        }),
        props: ['solution'],
        methods: {
            createMaze() {
                this.$emit('mazeData', {
                    columns: +this.width,
                    rows: +this.height,
                    startPoint: this.startPoint.split(','),
                    endPoint: this.endPoint.split(','),
                    brickDensity: +this.brickDensity})
            },
            regenerateMaze () {
                this.$emit('regenerateMaze');
            }
        }
    }
</script>

<style scoped>
    .form {
        display: flex;
        flex-direction: column;
        width: 300px;
        align-items: flex-start;
    }
    input {
        display: block;
    }
    button {

    }
</style>
