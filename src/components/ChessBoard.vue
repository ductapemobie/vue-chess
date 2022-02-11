<template>
    <div class="chess-grid">
        <div v-for="row in initialBoard" :key="row" class="chess-row">
            <div v-for="square in row.cells" :key="square" class="chess-square">
                <ChessSquare :x="square.x" :y="square.y" :color="square.color" :piece="square.piece" :border="square.border" @squareclicked="squareClicked"/>
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
                            {x:4, y:7, canCastle:"true"}
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
                            {x:0, y:7, canCastle:"true"},
                            {x:7, y:7, canCastle:"true"},
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
                            {x:4, y:0, canCastle:"true"}
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
                            {x:0, y:0, canCastle:"true"},
                            {x:7, y:0, canCastle:"true"}
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

                const squares = [];
                for (let i = 0 ; i < 8; i ++){
                    squares.push({row:i, cells:[]});
                    for (let j = 0; j < 8; j++){
                        const squareColor = (this.selectedPiece.x === j && this.selectedPiece.y === i)?
                            "red" : ((i+j) % 2 === 1)? 'blue' : 'white'
                        
                        const squarePiece = this.getSquarePiece(j, i, this.pieces);
                        squares[i].cells.push({y: i, x: j, color: squareColor, piece: squarePiece, border: false})
                    }
                }
                this.legalMoves.forEach(move=>{
                    squares[move.y].cells[move.x].border = true;
                })
                return squares;
            }
        },
        methods: {
            squareClicked(coords){
                this.selected.x = coords.x;
                this.selected.y = coords.y;
                const selectedPiece = coords.piece;
                this.processClick(selectedPiece);
            },
            processClick(selectedPiece){
                if (this.selectedPiece.piece && selectedPiece[0] != this.turn[0]){
                    this.legalMoves.forEach(move => {
                        if (move.x === this.selected.x && move.y === this.selected.y){
                            this.movePiece();
                        }
                    });
                    this.selectedPiece.x = -1;
                    this.selectedPiece.y = -1;
                    this.selectedPiece.piece = "";
                    this.legalMoves=[];
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
                const knightArr = [{x: -2, y:-1},{x: -2, y: 1},{x: -1, y:-2},{x: -1, y: 2},{x:  1, y:-2},{x:  1, y: 2},{x:  2, y:-1},{x:  2, y: 1}];
                const X = this.selectedPiece.x;
                const Y = this.selectedPiece.y;

                const moveList = [];

                switch (this.selectedPiece.piece[1]){
                    case 'k':{//TODO can't move into check
                        //console.log('k');
                        for (let i = -1 ; i < 2; i ++){
                            if (i + Y < 0)continue;
                            if (i + Y > 7)break;
                            for (let j = -1; j < 2; j++){
                                if (i === 0 && j === 0)continue;
                                if (j + X < 0)continue;
                                if (j + X > 7)break;
                                const destPiece = this.getSquarePiece(X + j, Y + i);
                                if (destPiece){
                                    if (destPiece[0] === this.turn[0])continue;
                                }
                                moveList.push({x: X + j, y: Y + i});
                            }
                            
                        }
                        {//TODO castle

                        }
                        break;
                    }
                    case 'q':{
                        //console.log('q');
                        for (let i = -1; i < 2; i++){
                            for (let j = -1; j < 2; j++){
                                if (i === 0 && j === 0) continue;
                                for (let k = 1; k < 8; k++){
                                    const destX = X + j * k;
                                    const destY = Y + i * k;
                                    if (destX > 7 || destX < 0 || destY > 7 || destY < 0)break;
                                    const destPiece = this.getSquarePiece(destX, destY);
                                    if (destPiece){
                                        if (destPiece[0] != this.turn[0]){
                                            moveList.push({x:destX, y:destY});
                                        }
                                        break;
                                    }
                                    moveList.push({x:destX, y:destY});
                                }
                            }
                        }
                        break;
                    }
                    case 'b':{
                        //console.log('b');
                        for (let i = -1; i < 2; i+=2){
                            for (let j = -1; j < 2; j+=2){
                                if (i === 0 && j === 0) continue;
                                for (let k = 1; k < 8; k++){
                                    const destX = X + j * k;
                                    const destY = Y + i * k;
                                    if (destX > 7 || destX < 0 || destY > 7 || destY < 0)break;
                                    const destPiece = this.getSquarePiece(destX, destY);
                                    if (destPiece){
                                        if (destPiece[0] != this.turn[0]){
                                            moveList.push({x:destX, y:destY});
                                        }
                                        break;
                                    }
                                    moveList.push({x:destX, y:destY});
                                }
                            }
                        }
                        break;
                    }
                    case 'n':{
                        //console.log('n');
                        for (let i = 0; i < 8; i++){
                            const destX = X + knightArr[i].x;
                            const destY = Y + knightArr[i].y;
                            if (destX > 7 || destX < 0 || destY > 7 || destY < 0)continue;
                            const destPiece = this.getSquarePiece(destX, destY);
                            if (destPiece){
                                if (destPiece[0] === this.turn[0])
                                    continue;
                            }
                            moveList.push({x:destX, y:destY});
                        }
                        break;
                    }  
                    case 'r':{
                        //console.log('r');
                        for (let i = -1; i < 2; i++){
                            for (let j = -1; j < 2; j++){
                                if (i === 0 && j === 0) continue;
                                if (i != 0 && j != 0) continue;
                                for (let k = 1; k < 8; k++){
                                    const destX = X + j * k;
                                    const destY = Y + i * k;
                                    if (destX > 7 || destX < 0 || destY > 7 || destY < 0)break;
                                    const destPiece = this.getSquarePiece(destX, destY);
                                    if (destPiece){
                                        if (destPiece[0] != this.turn[0]){
                                            moveList.push({x:destX, y:destY});
                                        }
                                        break;
                                    }
                                    moveList.push({x:destX, y:destY});
                                }
                            }
                        }
                        break;
                    }
                    case 'p':{
                        //console.log('p');
                        let direction = (this.turn === "white") ? -1 : 1;
                        {//checking directly in front
                            const destX = X;
                            const destY = Y + direction;
                            const destPiece = this.getSquarePiece(destX, destY);
                            if (!destPiece) {
                                moveList.push({x:destX, y:destY});
                                if ((this.turn === "white") ? (Y === 6) : (Y === 1)){
                                    const destX2 = X;
                                    const destY2 = destY + direction;
                                    const destPiece2 = this.getSquarePiece(destX2, destY2);
                                    if (!destPiece2)moveList.push({x:destX2, y:destY2});
                                }
                            }
                        }
                        {//checking captures
                            for (let i = -1; i < 2; i+=2){
                                const destX = X + i;
                                const destY = Y + direction;
                                const destPiece = this.getSquarePiece(destX, destY);
                                if (destPiece && destPiece[0] != this.turn[0])
                                    moveList.push({x:destX, y:destY});
                            }
                        }
                        {//TODO on croissant

                        }
                        //promotion handled in movePiece function
                        break;
                    }
                }
                this.legalMoves = moveList;
            },
            movePiece(){
                //1 capture if necessairy
                const newX = this.selected.x;
                const newY = this.selected.y;
                const turn = this.turn;
                const oppTurn = (turn === "white") ? "black" : "white";
                const capturedPiece = this.getSquarePiece(newX, newY, this.pieces);
                //a piece was captured
                if (capturedPiece){
                    const capPieceArr = this.pieces[oppTurn][capturedPiece[1]];
                    let capIndex = -1;
                    for (let i = 0; i < capPieceArr.length; i ++){
                        if (capPieceArr[i].x === newX &&
                            capPieceArr[i].y === newY){
                            capIndex = i;
                            break;
                        }
                    }
                    if (capIndex === -1) console.log("oh no");
                    this.pieces[oppTurn][capturedPiece[1]].splice(capIndex, 1);
                    //console.log(this.pieces);
                }

                //2 move the piece
                //2a identify piece
                const pieceType = this.selectedPiece.piece[1];
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

                //2d promote pawn if necessairy
                if (pieceType === "p"){
                    if ((this.turn === "white") ? (newY === 0) : (newY === 7)){
                        pieceArr.splice(index, 1);
                        this.pieces[turn]['q'].push({x:newX, y:newY});
                    }
                }

                //3 update turn
                this.turn = oppTurn;
            },
            getSquarePiece(x, y){
                function checkType(color, piece, pieces){
                    for (let i = 0; i < pieces[color][piece].length; i++)
                        if (pieces[color][piece][i].x === x && pieces[color][piece][i].y === y){
                            return true;
                        }
                    return false;
                }

                if (checkType('white', 'k', this.pieces)){
                    return 'wk';
                }else if(checkType('black', 'k', this.pieces)){
                    return 'bk';
                }else if(checkType('white', 'q', this.pieces)){
                    return 'wq'
                }else if(checkType('black', 'q', this.pieces)){
                    return 'bq'
                }else if(checkType('white', 'b', this.pieces)){
                    return 'wb'
                }else if(checkType('black', 'b', this.pieces)){
                    return 'bb'
                }else if(checkType('white', 'n', this.pieces)){
                    return 'wn'
                }else if(checkType('black', 'n', this.pieces)){
                    return 'bn'
                }else if(checkType('white', 'r', this.pieces)){
                    return 'wr'
                }else if(checkType('black', 'r', this.pieces)){
                    return 'br'
                }else if(checkType('white', 'p', this.pieces)){
                    return 'wp'
                }else if(checkType('black', 'p', this.pieces)){
                    return 'bp'
                }else{
                    return ''
                }
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