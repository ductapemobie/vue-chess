<template>

<table>
<tr>
    <th>Start pos</th>
    <th>End pos</th>
    <th>Moved Piece</th>
    <th>Captured Piece</th>
</tr>
<tr v-for:="(move, index) in moves" :key="move" v-bind:class="index % 2 === 0 ? 'white' : 'black'">
    <td>{{move.start.y}}{{move.start.x}}</td>
    <td>{{move.end.y}}{{move.end.x}}</td>
    <td>{{move.piece}}</td>
    <td>{{move.captured}}</td>  
</tr>

</table>

</template>

<script>
export default{
    name:"MovesTable",
    props:{
        moves: Array
    },
    computed:{

    }
    //maybe someday have a method to make moves look good
}
</script>

<style scoped>
    .white {
        background-color: #EEE;
        color: #111;
    }

    .black {
        background-color: #111;
        color: #eee;
    }


</style>