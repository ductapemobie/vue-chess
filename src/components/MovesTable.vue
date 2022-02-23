<template>

<table>
<tr>
    <th>White</th>
    <th>Black</th>
</tr>
<tr v-for:="(move, index) in formatMoves" :key="move">
    <td class="white">{{move.w}}</td>
    <td class="black">{{move.b}}</td>  
</tr>

</table>

</template>

<script>
export default{
    name:"MovesTable",
    props:{
        moves: {
            type:Array,
            default(rawProps){
                return [
                    {
                        start:{x: -1, y: -1},
                        end:{x: -1, y: -1},
                        piece:"ERROR",
                        captured:rawProps
                    }
                ]
            }
        
        }
    },
    computed:{
        formatMoves(){
            //TODO account for checks
            //TODO account for captures
            //TOTO account for castles
            //TODO account for 2 possible piece cases
            const yToRow = ['8', '7', '6', '5', '4', '3', '2', '1'];
            const xToCol = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
            const moveNotation =  this.moves.map(move => {
                if (move.castle){
                    return move.castle === 'right' ? "O-O" : "O-O-O";
                }
                let retStr = ""
                if (move.piece[1] != 'p'){
                    retStr = move.piece[1].toUpperCase();
                }
                
                if (move.captured){
                    retStr = retStr + 'x';
                }

                retStr = retStr + xToCol[move.end.x] + yToRow[move.end.y];

                return retStr;
            });

            const retArr = [];

            for (let i = 0 ; i < moveNotation.length; i+=2){
                const movePair = {
                    w: moveNotation[i],
                    b: moveNotation[i + 1]
                }
                retArr.push(movePair);
            }
            return retArr;
        }
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