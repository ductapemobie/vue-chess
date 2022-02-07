<template>
<div>
    <h1>grid test</h1>
    <div class="big-div" v-on:click="printGrid">
        <div v-for="row in grid" :key="row" class="row-div">
            <div v-for="square in row.squares" :key="square" class="small-div">
                {{square.x}} {{square.y}}
            </div>
        </div>
    </div>
</div>
</template>

<script>
export default {
    data(){
        return {

        }
    },
    computed:{
        grid(){
            const gridArr = [];
            for (let i = 0; i < 8; i++){
                gridArr.push({squares:[]})
                for (let j = 0; j < 8; j++){
                    gridArr[i].squares.push({x:j, y:i, color:(i+j%2) === 1 ? "white" : "blue"})
                }
            }
            return gridArr;
        }
    },
    methods:{
        printGrid(){
            console.log(this.grid);
        }
    }
}
</script>

<style scoped>

.big-div{
    width:40em;
    height:40em;
    background-color:green;
    margin: auto;
}

.small-div{
    width:5em;
    height:5em;
    background-color:white;
    display: inline-block;
}

.row-div{
    width:40em;
    height:5em;
    background-color:orange;
}

</style>