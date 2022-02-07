<template>
    <div class="chess-grid">
        <div v-for="row in initialBoard" :key="row" class="chess-row">
            <div v-for="square in row.cells" :key="square" class="chess-square">
                <ChessSquare :x="square.x" :y="square.y" :color="square.color" :piece="square.piece" @squareclicked="squareClicked"/>
            </div>
        </div>
    </div>
</template>

<script>
    import ChessSquare from "./ChessSquare.vue"

    export default {
        name: 'ChessBoard',
        components:{
            ChessSquare
        },
        data(){
            return {
                pieces:{
                    "white":{
                        "k":[
                            {x:4, y:7}
                        ],
                        "q":[
                            {x:3, y:7}
                        ],
                        "b":[
                            {x:2, y:7},
                            {x:5, y:7}
                        ],
                        "n":[
                            {x:1, y:7},
                            {x:6, y:7}
                        ],
                        "r":[
                            {x:0, y:7},
                            {x:7, y:7}
                        ],
                        "p":[
                            {x:0, y:6},
                            {x:1, y:6},
                            {x:2, y:6},
                            {x:3, y:6},
                            {x:4, y:6},
                            {x:5, y:6},
                            {x:6, y:6},
                            {x:7, y:6},
                        ]
                    },
                    "black":{
                        "k":[
                            {x:4, y:0}
                        ],
                        "q":[
                            {x:3, y:0}
                        ],
                        "b":[
                            {x:2, y:0},
                            {x:5, y:0}
                        ],
                        "n":[
                            {x:1, y:0},
                            {x:6, y:0}
                        ],
                        "r":[
                            {x:0, y:0},
                            {x:7, y:0}
                        ],
                        "p":[
                            {x:0, y:1},
                            {x:1, y:1},
                            {x:2, y:1},
                            {x:3, y:1},
                            {x:4, y:1},
                            {x:5, y:1},
                            {x:6, y:1},
                            {x:7, y:1},
                        ]
                    }
                },
                selected:{
                    x:0,
                    y:0,
                },
                turn: "white",
                selectedPiece: {
                    x: -1,
                    y: -1,
                    piece: ""
                },
                legalMoves:[]
            }
        },
        computed:{
            initialBoard(){

                //this function isn't great but oh well
                function getSquarePiece(x, y, pieces){

                    function checkType(color, piece){
                        for (let i = 0; i < pieces[color][piece].length; i++)
                            if (pieces[color][piece][i].x === x && pieces[color][piece][i].y === y){
                                return true;
                            }
                        return false;
                    }

                    if (checkType('white', 'k')){
                        return 'wk';
                    }else if(checkType('black', 'k')){
                        return 'bk';
                    }/*else if(checkType('white', 'q')){
                        return 'wq'
                    }else if(checkType('black', 'q')){
                        return 'bq'
                    }else if(checkType('white', 'b')){
                        return 'wb'
                    }else if(checkType('black', 'b')){
                        return 'bb'
                    }else if(checkType('white', 'k')){
                        return 'wn'
                    }else if(checkType('black', 'k')){
                        return 'bn'
                    }else if(checkType('white', 'r')){
                        return 'wr'
                    }else if(checkType('black', 'r')){
                        return 'br'
                    }else if(checkType('white', 'p')){
                        return 'wp'
                    }else if(checkType('black', 'p')){
                        return 'bp'
                    }*/else{
                        return ''
                    }
                }

                const squares = [];
                for (let i = 0 ; i < 8; i ++){
                    squares.push({row:i, cells:[]});
                    for (let j = 0; j < 8; j++){
                        const squareColor = (this.selected.x === j && this.selected.y === i)?
                            "red" : ((i+j) % 2 === 1)? 'blue' : 'white'
                        const squarePiece = getSquarePiece(j, i);
                        squares[i].cells.push({y: i, x: j, color: squareColor, piece: squarePiece})
                    }
                }
                return squares;
            }
        },
        methods: {
            squareClicked(coords){
                this.selected.x = coords.x;
                this.selected.y = coords.y;
                console.log(coords.x, coords.y)
                const selectedPiece = coords.piece;
                this.processClick(selectedPiece);
            },
            processClick(selectedPiece){
                if (this.selectedPiece.piece){
                    this.legalMoves.forEach(move => {
                        if (move.x === this.selected.x && move.y === this.selected.y){
                            this.movePiece();
                        }
                    });
                    this.selectedPiece.x = -1;
                    this.selectedPiece.y = -1;
                    this.selectedPiece.piece = "";
                    console.log(this.legalMoves);
                }else{
                    //unsleceted state
                    if (selectedPiece != ""){
                        const selectedColor = selectedPiece[0] === 'w' ? 'white' : 'black'
                        if (selectedColor === this.turn){
                            //store selected piece in state
                            this.selectedPiece.x = this.selected.x;
                            this.selectedPiece.y = this.selected.y;
                            this.selectedPiece.piece = selectedPiece;
                            this.determineLegalMoves();
                            return;
                        }else{
                            //cant select piece off their turn
                            return;
                        }
                    }else{
                        //no piece selected, no piece clicked => do nothing
                        return;
                    }
                }
            },
            determineLegalMoves(){
                //this is probably the biggest function
                //only to be called in the selected state
                if (this.selectedPiece.piece === "")return;
                
                const X = this.selectedPiece.x;
                const Y = this.selectedPiece.y;

                const moveList = [];

                switch (this.selectedPiece.piece[1]){
                    case 'k':
                        console.log('k');
                        for (let i = -1 ; i < 2; i ++){
                            if (i + Y < 0)continue;
                            if (i + Y > 7)break;
                            for (let j = -1; j < 2; j++){
                                if (i === 0 && j === 0)continue;
                                if (j + X < 0)continue;
                                if (j + X > 7)break;
                                moveList.push({x: X + j, y: Y + i});
                            }
                            this.legalMoves = moveList;
                        }
                        break;
                    case 'q':
                        console.log('q');
                        break;
                    case 'b':
                        console.log('b');
                        break;
                    case 'n':
                        console.log('k');
                        break;
                    case 'r':
                        console.log('r');
                        break;
                    case 'p':
                        console.log('p');
                        break;
                }
                console.log(X, Y)
                console.log(moveList);
                return moveList;
            },
            movePiece(){
                //1 capture if necessairy
                const newX = this.selected.x;
                const newY = this.selected.y;

                //2 move the piece
                //2a identify piece
                const pieceType = this.selectedPiece.piece[1];
                const turn = this.turn;
                const pieceArr = this.pieces[turn][pieceType]
                let index = -1;
                for (let i = 0; i < pieceArr.length; i ++){
                    if (pieceArr[i].x === this.selectedPiece.x &&
                        pieceArr[i].y === this.selectedPiece.y){
                        index = i;
                        break;
                    }
                }
                //2b sanity check
                if (index === -1) console.log("ERROR: piece list corrupted??")
                //2c update piece in pieces list
                pieceArr[index].x = this.selected.x;
                pieceArr[index].y = this.selected.y;
                //3 update turn
                this.turn = (turn === "white") ? "black" : "white";
            },
            checkType(color, piece){
                for (let i = 0; i < pieces[color][piece].length; i++)
                    if (this.pieces[color][piece][i].x === x && this.pieces[color][piece][i].y === y){
                        return true;
                    }
                return false;
            }
        }
    }
</script>

<style scoped>

.chess-grid{
    width: 40em;
    height: 40em;
    margin: auto;
}

.chess-row{
    height: 5em;
}

.chess-square{
    display: inline-flex;
}

</style>