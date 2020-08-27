<template>
    <div class="maze">
        <div class="row" v-for="row in mazeData.rows" :key="row" :style="{height: 700/mazeData.rows + 'px'}">
<!--                    :ref="solutionSquares && solutionSquares[row] ? (solutionSquares[row].indexOf(column) > -1) ? 'block'+row+column : null : null"-->
            <MazeBlock
                    v-for="column in mazeData.columns"
                    :key="column"
                    :style="{height: 700/mazeData.rows + 'px', width: 700/mazeData.columns+ 'px'}"
                    ref="block"
                    :class="{
                        start: mazeData.startPoint[0] == row && mazeData.startPoint[1] == column,
                        end: mazeData.endPoint[0] == row && mazeData.endPoint[1] == column,
                        brick: brickPositions[row] ? brickPositions[row].indexOf(column) > -1 : false,
                        roadToEnd: solutionSquares && solutionSquares[row] ? solutionSquares[row].indexOf(column) > -1 : false,
                        ifNoSolution: !solutionSquares ? (
                            !(mazeData.startPoint[0] == row && mazeData.startPoint[1] == column)
                            && !(mazeData.endPoint[0] == row && mazeData.endPoint[1] == column)
                            && (brickPositions[row] ? brickPositions[row].indexOf(column) < 0 : true)
                            ) : false
                    }">
            </MazeBlock>
        </div>
    </div>
</template>

<script>
    import MazeBlock from "./MazeBlock";
    export default {
        props: ['mazeData'],
        components: {MazeBlock},
        data: () => ({
            brickPositions: {},
            walkedPositions: [],
            allWalkedSquares: {},
            moves: [[1,0], [-1, 0], [0,1], [0,-1]],
            solutionSquares: null
        }),
        mounted() {
            var start = this.mazeData.startPoint;
            start[0] = +start[0]; //make them numbers
            start[1] = +start[1];
            var end = this.mazeData.endPoint;
            end[0] = +end[0];
            end[1] = +end[1];

            var rows = this.mazeData.rows;
            var columns = this.mazeData.columns;
            var density = 100/this.mazeData.brickDensity;
            var maxBricks = Math.floor(rows*columns / density); //nr of max bricks on the container
            var maze = [];
            var obj = {};

            for (let i = 1; i <= rows; i++) {//the matrix with all of the positions in the maze
                for (let j = 1; j <= columns; j++) {
                    maze.push([i, j]);
                }
            }

            var randomBricks = [...this.generateRandomBricks(maze, maxBricks, start, end)]; ////generate the positions of the bricks

            for (let brick of randomBricks) {
                if (!obj[brick[0]]) obj[brick[0]] = []
                obj[brick[0]].push(brick[1]) //object with key being = the row and value  = array with columns
            }

            for (let pos in Object.values(obj)) { //sort the columns
                if (obj[pos]) obj[pos].sort((a,b) => a-b);
            }

            this.brickPositions = {...obj};

            var solution = this.solution(maze, randomBricks, start, end);


            let ob2 = {}; //this will be the object containing all the routes that lead to the end
            ob2[0] = [[...this.allWalkedSquares[0][0].from]];

            //for-ul de sub compune obiectul ob2 ale carui valori vor fi toate drumurile intregi parcurse, patrat dupa patrat, pentru a gasi apoi printre ele ruta care duce la final
            for (let key of Object.keys(this.allWalkedSquares)) {

                let crtArr = this.allWalkedSquares[key];
                if (+key == 1) {
                    for (let c = 0; c < crtArr.length; c++) {
                        let prevArr = this.allWalkedSquares[+key-1];
                        // for (let h = 0; h < prevArr.length; h++) {
                         if (this.compareArrays(crtArr[c].from, prevArr[0].from)) {
                            if (!ob2[c]) ob2[c] = [[...this.allWalkedSquares[0][0].from]];
                            ob2[c].push(crtArr[c].to);
                         }
                    }
                }
                else if (+key > 1) {
                    let prevArr = this.allWalkedSquares[+key-1];
                    for (let p = 0; p < prevArr.length; p++) {
                        let repeat = 0;
                        let tmp = [];
                        let keyForTmp = null;
                        for (let c = 0; c < crtArr.length; c++) {
                             if (this.compareArrays(crtArr[c].from, prevArr[p].to)) {
                                 repeat++;
                                 for (let obKey of Object.keys(ob2)) {
                                      if (this.compareArrays(ob2[obKey][ob2[obKey].length-1], crtArr[c].from)) {
                                         if (repeat == 1) {
                                            tmp = [...ob2[obKey]];
                                            keyForTmp = obKey;
                                            ob2[obKey].push(crtArr[c].to);
                                            // break;
                                         }
                                     }
                                 }
                                if (repeat > 1 && keyForTmp) {
                                    let hh = null;
                                    for (let obKey of Object.keys(ob2)) {
                                        if (keyForTmp == obKey) {
                                            if (!ob2[+obKey+1]) {
                                            }
                                            else {
                                                hh = [...ob2[+obKey+1]];
                                            }
                                                ob2[+obKey+1] = [...tmp];
                                                ob2[+obKey+1].push(crtArr[c].to);
                                        }
                                        else if (obKey > keyForTmp+1) {
                                            if (!hh) return solution = 'A rare error which i didnt catch'
                                            if (ob2[obKey].length) {
                                                let hh2 = [...ob2[obKey]];
                                                ob2[obKey] = [...hh];
                                                hh = [...hh2];
                                                if (!ob2[+obKey+1]) {
                                                    ob2[+obKey + 1] = [...hh];
                                                }
                                            }
                                            else {
                                                ob2[obKey] = [...hh];
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
            if (solution == 'Maze has solution') {
                this.solutionSquares = [];
                for (let key of Object.keys(ob2)) {
                    let arr = ob2[key];
                    if (this.compareArrays(arr[arr.length-1], this.mazeData.endPoint)) {
                        if (!this.solutionSquares.length || arr.length < this.solutionSquares.length) {
                            this.solutionSquares = [...ob2[key]];
                        }
                    }
                }
                this.solutionSquares = this.solutionSquares.slice(1, this.solutionSquares.length-1);

                let temp = {};
                for (let sol of this.solutionSquares) {
                    if (!temp[sol[0]]) temp[sol[0]] = [];
                    temp[sol[0]].push(sol[1]);
                }
                this.solutionSquares= {};
                this.solutionSquares = {...temp};
            }
            this.$emit('solution', solution);
            this.addAnimation(this.solutionSquares);
        },
        methods: {
            generateRandomBricks(initialMaze, maxBricks, startPoint, endPoint) { //generate the positions of the bricks
                var arr = [];

                while (arr.length < maxBricks) { //if it isnt filled from the first while loop, go again
                    for (let square of initialMaze) {
                        if (
                            arr.indexOf(square) < 0 //if it hasnt beed added before
                            && this.getRandomInt(100) <= this.mazeData.brickDensity // % chance of brick
                            && !this.compareArrays(square, startPoint) // brick must not be on start position
                            && !this.compareArrays(square, endPoint) // brick must not be on end position
                        ) {
                            if (arr.length == maxBricks) return arr; // if reached the needed nr of bricks, exit
                            arr.push(square);
                        }
                    }
                }
                return arr;
            },
            getRandomInt(max) {
                return Math.floor(Math.random() * Math.floor(max+1));
            },
            compareArrays (arr1, arr2) {
                return arr1.every((el, i) => el == arr2[i]);
            },
            solution(maze, bricks, start, end) {
                var moves = [[1,0], [-1, 0], [0,1], [0,-1]]; //possible increments of the square positions
                var rightLimit = this.mazeData.columns;
                var bottomLimit = this.mazeData.rows;
                this.walkedPositions.push([...start]);


                if (this.isSurroundedByBricks(start, bricks, moves) || this.isSurroundedByBricks(end, bricks, moves)) {
                    return 'no solution';
                } // if start or end is surrounded by bricks, there is no solution

                var i = 0;
                this.allWalkedSquares[0] = [{from: [...start]}];

                while (true) {
                    i++;
                    let res = this.nextAvailableMoves(this.walkedPositions, moves, bricks, rightLimit, bottomLimit);
                    let next = res[0];

                    // console.log('res1', res[1]);

                    // this.allWalkedSquares[i] = [...next];
                    this.allWalkedSquares[i] = [...res[1]];
                    // console.log('allWalkedSquares', this.allWalkedSquares);



                    if (next.length == 0) return 'Maze has no solution';

                    this.walkedPositions.push(...next);
                    for (let pos of next) {
                        if (this.compareArrays(pos, end)) return 'Maze has solution'
                    }
                }
            },
            nextAvailableMoves(walkedPositions, moves, bricks, rightLimit, bottomLimit) {
                var nextPositions = [];
                var someth = [];
                for (let pos of walkedPositions) { //iterate through the walked positions array
                    for (let move of moves) { //and increment them
                        let arr = []
                        if (pos[0] + move[0] <= rightLimit //check that the next square doesnt exceed the limits of the container
                            && pos[0] + move[0] > 0
                            && pos[1] + move[1] <= bottomLimit
                            && pos[1] + move[1] > 0) {

                            arr = [pos[0] + move[0], pos[1]+move[1]]; //create the position of the square

                            let bricksExists = false;
                            for (let brick of bricks) {
                                if (this.compareArrays(brick, arr)) bricksExists = true; //check if a brick is on that position
                            }

                            let alreadyExists = false;
                            for (let st of walkedPositions) {
                                if (this.compareArrays(arr, st)) alreadyExists = true //check if position was generated before
                            }

                            if (!bricksExists && !alreadyExists) {
                                someth.push({to: arr, from: pos})
                                nextPositions.push(arr)

                            }
                        }
                    }
                }
                return [nextPositions, someth];
            },
            isSurroundedByBricks(position, bricks, moves) {
                for (let move of moves) {
                    let nextMove = [position[0] + move[0], position[1]+move[1]];
                    if (bricks.indexOf(nextMove) < 0) return false;
                }
                return true;
            },
            addAnimation(solutionSquares) {

            }
        }
    }
</script>

<style scoped>
    .maze {
        height: 700px;
        width: 700px;
        border: 1px solid black;
        /*display: flex;*/
        /*flex-wrap: wrap;*/
        justify-content: flex-start;
        align-items: flex-start;
    }
    .row {
        display: flex;
        /*flex-direction: row;*/
    }
    .end {
        background-color: lightcoral;
    }
    .start {
        background-color: lightgreen;
    }
    .brick {
        background-color: darkgray;
    }
    .roadToEnd {
        background-color: lightblue;
        -webkit-animation: fadein 0.5s; /* Safari, Chrome and Opera > 12.1 */
        -moz-animation: fadein 0.5s; /* Firefox < 16 */
        animation: fadein 0.5s;
    }
    .ifNoSolution {
        background-color: lightyellow;
    }

    @keyframes fadein {
        from {
            opacity: 0;
        }
        to   {
            opacity: 1;
        }
    }

    /* Firefox < 16 */
    @-moz-keyframes fadein {
        from { opacity: 0; }
        to   { opacity: 1; }
    }

    /* Safari, Chrome and Opera > 12.1 */
    @-webkit-keyframes fadein {
        from {
            opacity: 0;
        }
        to   {
            opacity: 1;
        }
    }
</style>


