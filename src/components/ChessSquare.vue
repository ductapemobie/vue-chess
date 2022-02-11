<template>
    <div v-bind:class="[color, borderStyle]" class="square"  v-on:click="squareClicked">
        <div v-if:="piece">
            <img v-bind:src="require(`@/assets/${piece}.png`)" class="img-format"/>
        </div>
        <div v-if:="piece=='ewqqfw'">
            <div>{{x}} {{y}}</div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'ChessSquare',
    props:{
        x: Number,
        y: Number,
        color: String,
        piece: String,
        border: Boolean,
    },
    computed:{
        borderStyle(){
            if(this.border)console.log()
            return (this.border?"border":"no-border");
        }
    },
    data(){
        return {
            gridStyle:{
                gridArea: `${this.x}${this.y}`
            }
        }
    },
    methods:{
        squareClicked(){
            this.$emit("squareclicked", {x:this.x, y:this.y, piece:this.piece})
        }
    }
}
</script>

<style scoped>
.blue{
    background-color:rgb(86, 86, 235);
}

.white{
    background-color:rgb(133, 206, 255);
}

.red{
    background-color:red;
}

.square{
    width: 5em;
    height: 5em;
    display: inline-block;
}

.img-format{
    width: 5em;
    height: 5em;
    -webkit-user-drag: none;
    user-select: none;
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
}

.border{
    box-shadow: 0em 0em 1em red inset;
}

.no-border{
    text-align: center;
}

</style>