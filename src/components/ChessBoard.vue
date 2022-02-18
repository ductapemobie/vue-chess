<template>
    <div>
    <div class="chess-grid">
        <div v-for="row in initialBoard" :key="row" class="chess-row">
            <div v-for="square in row.cells" :key="square" class="chess-square">
                <ChessSquare :x="square.x" :y="square.y" :color="square.color" :piece="square.piece" :border="square.border" @squareclicked="squareClicked"/>
            </div>
        </div>
    </div>
    <MovesTable :moves="moveHistory"/>
    </div>
</template>

<script>
    import ChessSquare from "./ChessSquare.vue"
    import MovesTable from "./MovesTable.vue"

    export default {
        name: 'ChessBoard',
        components:{
            ChessSquare,
            MovesTable
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
                legalMoves:[],
                boardState:[
                    ["br", "bn", "bb", "bq", "bk", "bb", "bn", "br"],
                    ["bp", "bp", "bp", "bp", "bp", "bp", "bp", "bp"],
                    ["", "", "", "", "", "", "", ""],
                    ["", "", "", "", "", "", "", ""],
                    ["", "", "", "", "", "", "", ""],
                    ["", "", "", "", "", "", "", ""],
                    ["wp", "wp", "wp", "wp", "wp", "wp", "wp", "wp"],
                    ["wr", "wn", "wb", "wq", "wk", "wb", "wn", "wr"],
                ],
                moveHistory:[]
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
                        
                        const squarePiece = this.boardState[i][j]//this.getSquarePiece(j, i, this.pieces);
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
                            this.movePiece(move);
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
                             const possibleMoves = this.determineLegalMoves();
                             this.legalMoves = this.determineCheck(possibleMoves);
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
                //this might not be the biggest function anymore
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
                                const destPiece = this.boardState[Y + i][X + j];
                                if (destPiece){
                                    if (destPiece[0] === this.turn[0])continue;
                                }
                                moveList.push({x: X + j, y: Y + i});
                            }
                        }
                        //castle
                        //only 1 king ever i think, could change to be more robust though
                        if (this.pieces[this.turn]['k'][0].canCastle){
                            const backRow = this.turn === 'white' ? 7 : 0
                            //left castle
                            {
                                this.pieces[this.turn]['r'].forEach(rook=>{
                                    if (!rook.canCastle)return;
                                    if (rook.x === 0 && rook.y === (backRow)){
                                        let leftCastle = true;
                                        for (let i = 1; i < 4; i++){
                                            if (this.boardState[backRow][i])leftCastle = false;
                                        }
                                        if (leftCastle)moveList.push({x:2, y:backRow, castle:"left"})
                                    }
                                })
                            }
                            //right castle
                            {
                                this.pieces[this.turn]['r'].forEach(rook=>{
                                    if (rook.x === 7 && rook.y === (this.turn === 'white' ? 7 : 0)){
                                        let rightCastle = true;
                                        for (let i = 5; i < 7; i++){
                                            if (this.boardState[backRow][i])rightCastle = false;
                                        }
                                        if (rightCastle)moveList.push({x:6, y:backRow, castle:"right"})
                                    }
                                })
                            }
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
                                    const destPiece = this.boardState[destY][destX];
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
                                    const destPiece = this.boardState[destY][destX];
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
                            const destPiece = this.boardState[destY][destX];
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
                                    const destPiece = this.boardState[destY][destX];
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
                            const destPiece = this.boardState[destY][destX];
                            if (!destPiece) {
                                moveList.push({x:destX, y:destY});
                                if ((this.turn === "white") ? (Y === 6) : (Y === 1)){
                                    const destX2 = X;
                                    const destY2 = destY + direction;
                                    const destPiece2 = this.boardState[destY2][destX2];
                                    if (!destPiece2)moveList.push({x:destX2, y:destY2});
                                }
                            }
                        }
                        {//checking captures
                            for (let i = -1; i < 2; i+=2){
                                const destX = X + i;
                                const destY = Y + direction;
                                const destPiece = this.boardState[destY][destX];
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
                return moveList;
            },
            movePiece(move){
                //1 capture if necessairy
                const X = this.selectedPiece.x;
                const Y = this.selectedPiece.y;
                const newX = move.x;
                const newY = move.y;
                const turn = this.turn;
                const oppTurn = (turn === "white") ? "black" : "white";
                const backRow = (this.turn === 'white') ? 7 : 0;
                const capturedPiece = this.boardState[newY][newX];
                //will need to add some goofy stuff to this for en passant promotion and castles
                const moveObj = {
                    captured: capturedPiece,
                    piece: this.selectedPiece.piece,
                    start: {x : X, y : Y},
                    end: {x : newX, y : newY}
                };
                this.moveHistory.push(moveObj);
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
                if (pieceArr[index].canCastle)pieceArr[index].canCastle = false;

                //2d update boardstate
                this.boardState[newY][newX] = this.selectedPiece.piece;
                this.boardState[Y][X] = "";

                //2e promote pawn if necessairy
                if (pieceType === "p"){
                    if ((this.turn === "white") ? (newY === 0) : (newY === 7)){
                        pieceArr.splice(index, 1);
                        this.pieces[turn]['q'].push({x:newX, y:newY});
                        this.boardState[newY][newX] = turn[0] + 'q' 
                    }
                }

                //2f check castle case
                if (move.castle){
                    if (move.castle === "right"){
                        this.pieces[turn]['r'].forEach( rook => {
                            if (rook.x === 7 && rook.y === backRow){
                                rook.x = 5;
                                rook.canCastle = false;
                                this.boardState[backRow][7] = "";
                                this.boardState[backRow][5] = turn[0] + 'r'
                            }
                        })
                    }else{//left castle
                        this.pieces[turn]['r'].forEach( rook => {
                            if (rook.x === 0 && rook.y === backRow){
                                rook.x = 3;
                                rook.canCastle = false;
                                this.boardState[backRow][0] = "";
                                this.boardState[backRow][3] = turn[0] + 'r'
                            }
                        })
                    }
                }

                //3 update turn
                this.turn = oppTurn;
            },
            determineCheck(possibleMoves){
                //called to filter legalMoves to make sure u dont put urself in check
                
                const turn = this.turn;
                const oppTurn = (turn === 'white') ? 'black' : 'white';
                const X = this.selectedPiece.x;
                const Y = this.selectedPiece.y;
                const oppDirection = (turn === 'white') ? 1 : -1;
                const piece = this.selectedPiece.piece;
                let kingSquare = this.pieces[turn]['k'][0];
                const copiedState = this.boardState.map(row=>row.map(cell => cell));

                return possibleMoves.filter(move => {
                    //move holds {x, y} of possible dest square
                    const takenPiece = copiedState[move.y][move.x]
                    copiedState[move.y][move.x] = piece;
                    copiedState[Y][X] = "";
                    let inCheck = false;
                    

                    if (piece[1] === 'k'){
                        kingSquare = move;
                    }

                    const boardPieces = {};

                    for (let i = 0; i < 8; i++){
                        for (let j = 0; j < 8; j ++){
                            const piece = copiedState[i][j];
                            if (piece){
                                if (piece[0] === oppTurn[0]){
                                    if (!boardPieces[piece[1]]){
                                        boardPieces[piece[1]] = [];
                                    }
                                    boardPieces[piece[1]].push({x: j, y: i});
                                }
                            }
                        }
                    }

                    if (boardPieces['p']){
                        boardPieces['p'].forEach((pawn) => {
                            if (
                                (pawn.x + 1 === kingSquare.x || pawn.x - 1 === kingSquare.x) &&
                                (pawn.y + oppDirection === kingSquare.y)
                            ){
                                inCheck = true;
                                return;
                            }
                        })
                    }

                    if (boardPieces['n']){
                        const knightArr = [
                            {x: -2, y:-1},{x: -2, y: 1},
                            {x: -1, y:-2},{x: -1, y: 2},
                            {x:  1, y:-2},{x:  1, y: 2},
                            {x:  2, y:-1},{x:  2, y: 1}
                        ];

                        boardPieces['n'].forEach((knight) => {
                            if (
                                knightArr.reduce((status, move) => {
                                    const moveCoords = {
                                        x: knight.x + move.x,
                                        y: knight.y + move.y
                                    }

                                    if (
                                        (moveCoords.x === kingSquare.x) && (moveCoords.y === kingSquare.y)
                                    ){
                                        return true;
                                    }
                                    return status;
                                }, false)
                            ){
                                inCheck = true;
                                return;
                            }
                        });
                    }

                    if (boardPieces['k']){
                        const kingArr = [
                            {x: -1, y: -1},{x: -1, y:  0},{x: -1, y:  1},
                            {x:  0, y: -1},{x:  0, y:  1},
                            {x:  1, y: -1},{x:  1, y:  0},{x:  1, y:  1}
                        ]
                        
                        boardPieces['k'].forEach((king) => {
                            if (
                                kingArr.reduce((status, move) => {
                                    const moveCoords = {
                                        x: king.x + move.x,
                                        y: king.y + move.y
                                    }
                                    if (
                                        (moveCoords.x === kingSquare.x) && (moveCoords.y === kingSquare.y)
                                    ){
                                        return true;
                                    }
                                    return status;
                                }, false)
                            ){
                                inCheck = true;
                                return;
                            }
                        })
                    }

                    if (boardPieces['q']){
                        boardPieces['q'].forEach(queen => {
                            for (let i = -1; i < 2; i++){
                                for (let j = -1; j < 2; j++){
                                    if (i === 0 && j === 0)continue;
                                    for (let k = 1; k < 8; k++){
                                        const moveCoords = {
                                            x: queen.x + j * k,
                                            y: queen.y + i * k
                                        }
                                        if (moveCoords.x < 0 || moveCoords.x > 7 || moveCoords.y < 0 || moveCoords.y > 7)break;
                                        const pieceAtMove = copiedState[moveCoords.y][moveCoords.x];
                                        if (pieceAtMove){
                                            if (moveCoords.x === kingSquare.x && moveCoords.y === kingSquare.y){
                                                inCheck = true;
                                                return;
                                            }
                                            break;
                                        }
                                    }
                                }
                            }
                        })
                    }

                    if (boardPieces['b']){
                        boardPieces['b'].forEach(bishop => {
                            for (let i = -1; i < 2; i+=2){
                                for (let j = -1; j < 2; j+=2){
                                    if (i === 0 && j === 0)continue;
                                    for (let k = 1; k < 8; k++){
                                        const moveCoords = {
                                            x: bishop.x + j * k,
                                            y: bishop.y + i * k
                                        }
                                        if (moveCoords.x < 0 || moveCoords.x > 7 || moveCoords.y < 0 || moveCoords.y > 7)break;
                                        const pieceAtMove = copiedState[moveCoords.y][moveCoords.x];
                                        if (pieceAtMove){
                                            if (moveCoords.x === kingSquare.x && moveCoords.y === kingSquare.y){
                                                inCheck = true;
                                                return;
                                            }
                                            break;
                                        }
                                    }
                                }
                            }
                        });
                    }

                    if (boardPieces['r']){
                        boardPieces['r'].forEach(rook => {
                            for (let i = -1; i < 2; i++){
                                for (let j = -1; j < 2; j++){
                                    if (i === 0 && j === 0)continue;
                                    if (i != 0 && j != 0)continue;
                                    for (let k = 1; k < 8; k++){
                                        const moveCoords = {
                                            x: rook.x + j * k,
                                            y: rook.y + i * k
                                        }
                                        if (moveCoords.x < 0 || moveCoords.x > 7 || moveCoords.y < 0 || moveCoords.y > 7)break;
                                        const pieceAtMove = copiedState[moveCoords.y][moveCoords.x];
                                        if (pieceAtMove){
                                            if (moveCoords.x === kingSquare.x && moveCoords.y === kingSquare.y){
                                                inCheck = true;
                                                return;
                                            }
                                            break;
                                        }
                                    }
                                }
                            }
                        })
                    }

                    copiedState[Y][X] = piece;
                    copiedState[move.y][move.x] = takenPiece;
                    return (inCheck === false);

                })
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