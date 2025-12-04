<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Chess Puzzles — Local & vs Computer</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/css/chessboard.min.css" integrity="sha512-+q2xg5r3rg3r0r2u6ja0j2sZQm4u3q6q9nXq3v0s4vYv2ZbP0l2h9Q3k2W4l1sGm5k6h7n8o9p0q1r2s3t4u==" crossorigin="anonymous" referrerpolicy="no-referrer" />
  <style>
    body{font-family:Inter,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial;margin:0;padding:0;background:#f4f6f8;color:#111}
    header{background:#0b213f;color:#fff;padding:18px 24px;display:flex;align-items:center;justify-content:space-between}
    header h1{margin:0;font-size:18px}
    .container{max-width:1100px;margin:24px auto;padding:12px}
    .layout{display:grid;grid-template-columns:420px 1fr;gap:18px}
    .card{background:#fff;padding:14px;border-radius:10px;box-shadow:0 6px 18px rgba(12,30,50,0.08)}
    #board{width:400px;margin:0 auto}
    .controls{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
    button{padding:8px 12px;border-radius:8px;border:0;background:#0b6cff;color:#fff;cursor:pointer}
    button.secondary{background:#e6eefc;color:#0b213f}
    .puzzle-list{max-height:280px;overflow:auto;margin-top:8px}
    .puzzle-item{padding:8px;border-radius:6px;cursor:pointer}
    .puzzle-item:hover{background:#f1f7ff}
    .status{margin-top:10px;font-weight:600}
    .log{height:160px;overflow:auto;border:1px dashed #e0e6ef;padding:8px;border-radius:6px;background:#fbfdff}
    label{display:block;font-size:13px;margin-bottom:6px}
    .top-controls{display:flex;gap:8px;align-items:center}
    footer{padding:12px;text-align:center;color:#666;font-size:13px}
    @media (max-width:900px){.layout{grid-template-columns:1fr}#board{width:320px}}
  </style>
</head>
<body>
  <header>
    <h1>Chess Puzzles — Play locally or vs Computer</h1>
    <div>Single file demo • Local & AI play</div>
  </header>

  <div class="container">
    <div class="layout">
      <div class="card">
        <div id="board"></div>

        <div class="controls">
          <button id="newGameBtn">New Game</button>
          <button id="twoPlayerBtn" class="secondary">Two Players (Local)</button>
          <button id="vsComputerBtn">Play vs Computer</button>
          <button id="nextPuzzleBtn" class="secondary">Next Puzzle</button>
        </div>

        <div class="status" id="status">Status: Ready</div>
        <div style="margin-top:10px">
          <label>Difficulty (AI depth)</label>
          <select id="aiDepth"><option value="1">1 (Fast)</option><option value="2">2</option><option value="3" selected>3 (Balanced)</option><option value="4">4 (Slow)</option></select>
        </div>

        <div style="margin-top:10px">
          <label>Puzzle Controls</label>
          <div style="display:flex;gap:8px;align-items:center">
            <button id="checkSolutionBtn" class="secondary">Check Solution</button>
            <button id="showHintBtn" class="secondary">Show Hint</button>
          </div>
        </div>

        <div style="margin-top:10px">
          <label>Move Log</label>
          <div class="log" id="moveLog"></div>
        </div>

      </div>

      <div class="card">
        <h3>Chess Puzzles</h3>
        <p>Choose a puzzle to load a position. The goal is shown next to each puzzle. You can play two players locally, or try to solve puzzles vs the built-in AI.</p>

        <div class="puzzle-list" id="puzzleList"></div>

        <hr style="margin:12px 0">
        <h4>Quick Tips</h4>
        <ul>
          <li>Drag & drop pieces on the board to move.</li>
          <li>Use "Two Players" for local hotseat play.</li>
          <li>Use "Play vs Computer" — AI plays automatically after your move.</li>
          <li>Press "Next Puzzle" to cycle through preset puzzles.</li>
        </ul>
      </div>
    </div>
  </div>

  <footer>Made with ♟️ — Demo website. Copy and paste to run locally.</footer>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/1.0.0/chess.min.js" integrity="sha512-o4cXxYfCk+J6q3b9Xj3Q8r2a8D9j3m1sZ0q1w2e3r4t5y6u7i8o9p0q1r2s3t4u5" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/js/chessboard.min.js" integrity="sha512-3k2h4g5j6k7l8m9n0o1p2q3r4s5t6u7v8w9x0y1z2a3b4c5d6e7f8g9h0i1j2k3" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

  <script>
    // --- Basic app wiring ---
    const boardElement = document.getElementById('board');
    const statusEl = document.getElementById('status');
    const moveLogEl = document.getElementById('moveLog');
    const puzzleListEl = document.getElementById('puzzleList');

    // chess.js game object
    let game = new Chess();
    let board = null;
    let currentMode = 'two-player'; // 'two-player' | 'vs-computer' | 'puzzle'
    let aiColor = 'b';

    // Simple puzzle set: each puzzle has fen, goal, solution (array of SAN or UCI moves for checking).
    const PUZZLES = [
      {id:1, title:'Mate in 1 (White)', fen:'6k1/5pp1/8/8/8/8/5PPP/6K1 w - - 0 1', goal:'Deliver mate in 1', solution:['g2-g4']},
      {id:2, title:'Mate in 1 (Black to move)', fen:'6k1/5pp1/8/8/8/8/5PPP/6K1 b - - 0 1', goal:'Black mates in 1', solution:['g7-g6']},
      {id:3, title:'Classic Tactic', fen:'r1bqkbnr/pppp1ppp/2n5/4p3/3PP3/5N2/PPP2PPP/RNBQKB1R w KQkq - 2 4', goal:'Win material', solution:['d4xe5']},
      {id:4, title:'Fork tactic', fen:'rnbqkbnr/pppp1ppp/8/4p3/3P4/5N2/PPP1PPPP/RNBQKB1R b KQkq - 0 2', goal:'Find a fork or tactical win', solution:[]},
      {id:5, title:'Endgame (Pawn race)', fen:'8/8/8/8/4k3/8/4P3/4K3 w - - 0 1', goal:'Promote or win pawn race', solution:[]}
    ];

    let currentPuzzleIndex = 0;

    // --- UI population ---
    function populatePuzzleList(){
      puzzleListEl.innerHTML = '';
      PUZZLES.forEach((p, idx)=>{
        const div = document.createElement('div');
        div.className = 'puzzle-item';
        div.innerHTML = `<strong>${p.title}</strong><div style="font-size:13px;color:#555">${p.goal}</div>`;
        div.onclick = ()=>{loadPuzzle(idx)};
        puzzleListEl.appendChild(div);
      });
    }

    // --- Board config ---
    const cfg = {
      draggable: true,
      position: 'start',
      onDrop: onDrop,
      onSnapEnd: onSnapEnd,
      onMouseoverSquare: null,
      onMouseoutSquare: null,
    };

    board = Chessboard(boardElement, cfg);

    function setStatus(text){ statusEl.textContent = 'Status: ' + text; }

    // --- Move handling ---
    function onDrop(source, target, piece, newPos, oldPos, orientation){
      // Try making move on chess.js
      const move = game.move({from: source, to: target, promotion: 'q'});
      if(move === null){
        return 'snapback';
      } else {
        updateMoveLog();
        if(currentMode === 'vs-computer'){
          // After player's move, let computer play if it's its turn
          window.setTimeout(() => {
            if(!game.game_over() && game.turn() === aiColor){
              makeBestAIMove();
            }
          }, 250);
        }

        if(currentMode === 'puzzle'){
          checkPuzzleProgress();
        }
      }
    }

    function onSnapEnd(){
      board.position(game.fen());
    }

    function updateMoveLog(){
      moveLogEl.textContent = game.history({verbose:true}).map(m => m.san).join('  ');
      if(game.in_check()){
        setStatus((game.turn()==='w'?'White':'Black') + ' to move — Check!');
      } else if(game.game_over()){
        setStatus('Game over');
      } else {
        setStatus((game.turn()==='w'?'White':'Black') + ' to move');
      }
    }

    // --- New Game ---
    document.getElementById('newGameBtn').onclick = ()=>{
      game = new Chess();
      board.start();
      currentMode = 'two-player';
      setStatus('New game — Two player');
      updateMoveLog();
    }

    document.getElementById('twoPlayerBtn').onclick = ()=>{
      currentMode = 'two-player';
      setStatus('Two player mode');
    }

    document.getElementById('vsComputerBtn').onclick = ()=>{
      currentMode = 'vs-computer';
      // choose color: prompt quick
      const choose = confirm('OK = Play as White (computer Black). Cancel = Play as Black (computer White).');
      if(choose){ aiColor = 'b'; } else { aiColor = 'w'; }
      setStatus('Playing vs computer — AI plays ' + (aiColor==='w'?'White':'Black'));
      // if AI is white, let it play first
      if(game.turn() === aiColor){ makeBestAIMove(); }
    }

    // --- Simple AI (minimax with evaluate) ---
    function makeBestAIMove(){
      const depth = parseInt(document.getElementById('aiDepth').value,10) || 2;
      const best = minimaxRoot(depth, aiColor === 'w');
      if(best && best.length){
        game.move({from:best[0].from, to:best[0].to, promotion: 'q'});
        board.position(game.fen());
        updateMoveLog();
      }
    }

    // Minimax root
    function minimaxRoot(depth, isWhite){
      const moves = game.moves({verbose:true});
      let bestMove = null;
      let bestScore = isWhite ? -99999 : 99999;
      for(let i=0;i<moves.length;i++){
        const move = moves[i];
        game.move(move);
        const value = minimax(depth-1, -100000, 100000, !isWhite);
        game.undo();
        if(isWhite && value > bestScore){ bestScore = value; bestMove = move; }
        if(!isWhite && value < bestScore){ bestScore = value; bestMove = move; }
      }
      return bestMove ? [bestMove] : null;
    }

    function minimax(depth, alpha, beta, isWhite){
      if(depth === 0) return -evaluateBoard();
      const moves = game.moves({verbose:true});
      if(isWhite){
        let maxEval = -99999;
        for(const m of moves){
          game.move(m);
          const evalVal = minimax(depth-1, alpha, beta, false);
          game.undo();
          if(evalVal > maxEval) maxEval = evalVal;
          if(evalVal > alpha) alpha = evalVal;
          if(beta <= alpha) break;
        }
        return maxEval;
      } else {
        let minEval = 99999;
        for(const m of moves){
          game.move(m);
          const evalVal = minimax(depth-1, alpha, beta, true);
          game.undo();
          if(evalVal < minEval) minEval = evalVal;
          if(evalVal < beta) beta = evalVal;
          if(beta <= alpha) break;
        }
        return minEval;
      }
    }

    // Very simple material evaluation
    function evaluateBoard(){
      const values = {p:100, n:320, b:330, r:500, q:900, k:20000};
      const fen = game.fen();
      let score = 0;
      for(const c of fen){
        if(c === ' ') break;
        if(c === '/') continue;
        if(c >= '1' && c <= '8') continue;
        if(c.toLowerCase() === c){ // black piece
          score -= values[c];
        } else { // white piece
          score += values[c.toLowerCase()];
        }
      }
      return score;
    }

    // --- Puzzle loading & checking ---
    function loadPuzzle(idx){
      const p = PUZZLES[idx];
      currentPuzzleIndex = idx;
      game = new Chess(p.fen);
      board.position(game.fen());
      currentMode = 'puzzle';
      setStatus('Puzzle loaded: ' + p.title + ' — ' + p.goal);
      updateMoveLog();
    }

    document.getElementById('nextPuzzleBtn').onclick = ()=>{
      currentPuzzleIndex = (currentPuzzleIndex + 1) % PUZZLES.length;
      loadPuzzle(currentPuzzleIndex);
    }

    document.getElementById('checkSolutionBtn').onclick = ()=>{
      const p = PUZZLES[currentPuzzleIndex];
      if(!p.solution || p.solution.length===0){ alert('No canonical solution provided for this puzzle. Try to solve or check tactics manually.'); return; }
      // check if history contains solution prefix
      const hist = game.history();
      const sols = p.solution.map(m => m.replace(/-/g,'').replace(/x/g,''));
      // We'll check UCI-like from->to form for simplicity: convert san in solution? If solution provided as UCI/long algebraic it's easier.
      // For our demo we will check last moves match the provided moves (san). This is best-effort.
      const lastMoves = hist.slice(-p.solution.length);
      let ok = true;
      for(let i=0;i<p.solution.length;i++){
        // Compare by SAN (best-effort)
        // recreate move SAN by making a temp game from puzzle start
        const temp = new Chess(PUZZLES[currentPuzzleIndex].fen);
        const movesToCompare = hist.slice(0, i+1);
        // make i moves on temp
        for(let j=0;j<i+1;j++){
          // skip — we will compare current move only
        }
        // simple check: compare lastMoves[i] SAN to provided solution[i]
        if(!lastMoves[i] || lastMoves[i].toLowerCase() !== p.solution[i].toLowerCase()) { ok = false; break; }
      }
      if(ok) alert('Puzzle solved! (matches recorded solution)'); else alert('Solution does not match the recorded canonical solution. Keep trying!');
    }

    document.getElementById('showHintBtn').onclick = ()=>{
      const p = PUZZLES[currentPuzzleIndex];
      if(!p.solution || p.solution.length===0){ alert('No hint available for this puzzle.'); return; }
      alert('Hint — first move (SAN): ' + p.solution[0]);
    }

    function checkPuzzleProgress(){
      const p = PUZZLES[currentPuzzleIndex];
      if(!p.solution || p.solution.length===0) return;
      const hist = game.history();
      // Check if the last N moves match solution
      const N = p.solution.length;
      if(hist.length < N) return;
      const last = hist.slice(hist.length - N);
      let ok = true;
      for(let i=0;i<N;i++){
        if(last[i].toLowerCase() !== p.solution[i].toLowerCase()) { ok=false; break; }
      }
      if(ok){
        setStatus('Puzzle solved!');
        alert('Congratulations — You solved the puzzle!');
      }
    }

    // --- initialize ---
    populatePuzzleList();
    // load first puzzle by default
    loadPuzzle(0);

  </script>
</body>
</html>
