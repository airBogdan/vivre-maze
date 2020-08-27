<template>
    <div id="app">
        <Form @mazeData="createMaze" @regenerateMaze="regenerateMaze" :solution="solution"/>
        <Maze v-if="showMaze" :mazeData="mazeData" @solution="setSolution"/>
    </div>
</template>

<script>
import Form from "./components/Form";
import Maze from "./components/Maze";

export default {
    name: 'App',
    components: { Form, Maze },
    data: () => ({
        mazeData: {},
        showMaze: false,
        solution: null
    }),
    methods: {
        createMaze(data) {
            if (!Object.keys(this.mazeData).length) this.mazeData = {...data};
            this.showMaze = true;
        },
        regenerateMaze() {
            this.showMaze = false;
            setTimeout(()=> this.createMaze(), 0);
        },
        setSolution(sol) {
            this.solution = sol;
        }
    }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

    display: flex;
}
</style>
