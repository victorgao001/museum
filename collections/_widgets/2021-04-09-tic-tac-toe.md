---
layout: widget-base2
---

<div id="tictactoecontainer">
	<div id="game">
		<div id="status"></div>
		<div id="tttboard"></div>
		<div id="game-info">
			<button onClick="restart()">Restart</button>
			<button onClick="undo()">Undo</button>
			<button onClick="redo()">Redo</button>
		</div>
		<div>
			Mode:&nbsp;&nbsp;
			<select id="computerMove" value="0" onChange="setComputerMove(value)">
				<option value="0">Move First (X)</option>
				<option value="1">Move Second (O)</option>
				<option value="-1">Two Players</option>
			</select>
			Difficulty:
			<select id="difficulty" value="2" onChange="setDifficulty(value)">
				<option value="1">Easiest</option>
				<option value="2">Easy</option>
				<option value="3">Medium</option>
				<option value="4">Hard</option>
				<option value="5">Hardest</option>
			</select>
		</div>
	</div>
</div>

A tic tac toe game with several difficulties. On the hardest difficulty it is impossible to win, while on the easiest it is impossible to lose. This is possible because there are  relatively few options to play and the computer can just try them all.

Trivia
- There are 255 168 possible different tic tac toe games that can be played.
- While there are 19683 combinations of Xs and Os, only 5478 of them are reachable during a game.
- These boards can be represented as rotations or reflections of only 765 unique orbits.
- Open the web console to see possible moves the computer considers.
<style>
	.tttcell{
		width:75px;
		height:75px;
		border:1px solid black;
		display:inline-block;
		font-size:50px;
			text-align: center;
			vertical-align: top;
	}
	#tttboard{
		width:231px;
	}
</style>
<script>
	let LEN = 3, SZ = LEN * LEN;
	var win = [], lose = [], nxt = Array(19683).fill().map(() => Array(0));
	function hash(cur) {
		var ret = 0;
		for (let i = 0; i < SZ; i++) {
			ret = ret * 3;
			if (cur[i] === 'X') ret++;
			else if (cur[i] === 'O') ret += 2;
			else if (cur[i] !== null) ret += cur[i];
		}
		return ret;
	}
	function calc(cur, v, move) {
		if (win[v] != null) return;
		win[v] = lose[v] = 1000;
		var w = calculateWinner(cur);
		if (w != null) {
			win[v] = 100; lose[v] = -100;
			return;
		}
		for (let i = 0; i < 9; i++) if (cur[i] === null) {
			cur[i] = move;
			let x = hash(cur);
			if (win[x] == null) calc(cur, x, 3 - move);
			nxt[v].push([i, x]);
			win[v] = Math.min(win[v], -win[x]);
			lose[v] = Math.min(lose[v], -lose[x]);
			cur[i] = null;
		}
		if (win[v] === 1000) win[v] = 0;
		if (win[v] < 0) win[v]++;
		if (lose[v] === 1000) lose[v] = 0;
	}

	function calculateWinner(board) {
		for (let i = 0; i < LEN; i++) {
			if (board[i] && board[i] === board[i + LEN] && board[i] === board[i + 2 * LEN]) return board[i];
			if (board[i * LEN] && board[i * LEN] === board[i * LEN + 1] && board[i * LEN] === board[i * LEN + 2]) return board[i * LEN];
		}
		if (board[0] && board[0] === board[4] && board[0] === board[8]) return board[0];
		if (board[2] && board[2] === board[4] && board[2] === board[6]) return board[2];
		return null;
	}

	calc(Array(9).fill(null), 0, 1);
	let history = [
		{
			squares: Array(9).fill("")
		}
	];
	let stepNumber = 0;
	let xIsNext = true;
	let difficulty = 2;
	let computerMove = 0;

	function move(i) {
		history = history.slice(0, stepNumber + 1);
		current = history[history.length - 1];
		squares = current.squares.slice();
		if (calculateWinner(squares) || squares[i]) {
			return;
		}
		squares[i] = xIsNext ? "X" : "O";
		history = history.concat([
			{
				squares: squares
			}
		]);
		stepNumber = history.length - 1;
		xIsNext = !xIsNext;
		updateboard();
	}
	function handleClick(i) {
		if (xIsNext == computerMove) return;
		move(i);
		computerMove1();
	}
	function sleep(delay) {
		return new Promise(resolve => setTimeout(resolve, delay));
	}
	async function computerMove1() {
		await sleep(500);
		if (xIsNext != computerMove || stepNumber == SZ) return;
		history = history.slice(0, stepNumber + 1);
		current = history[history.length - 1];
		squares = current.squares.slice();
		var x = hash(squares), moves = [];
		if (difficulty == 1) {
			for (let i = 0; i < nxt[x].length; i++)
				if (lose[nxt[x][i][1]] > 0)
					moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (lose[nxt[x][i][1]] >= 0)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					moves.push(nxt[x][i][0]);
		}
		else if (difficulty == 2) {
			for (let i = 0; i < nxt[x].length; i++)
				moves.push(nxt[x][i][0]);
		}
		else if (difficulty == 3) {
			for (let i = 0; i < nxt[x].length; i++)
				if (win[nxt[x][i][1]] == 100)
					moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (win[nxt[x][i][1]] > -99)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					moves.push(nxt[x][i][0]);
		}
		else if (difficulty == 4) {
			for (let i = 0; i < nxt[x].length; i++)
				if (win[nxt[x][i][1]] >= 99)
					moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (win[nxt[x][i][1]] >= 0 || Math.random() < 0.2 && win[nxt[x][i][1]] > -98)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (win[nxt[x][i][1]] > -98)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (win[nxt[x][i][1]] > -99)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					moves.push(nxt[x][i][0]);
		}
		else if (difficulty == 5) {
			for (let i = 0; i < nxt[x].length; i++)
				if (win[nxt[x][i][1]] > 0)
					moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					if (win[nxt[x][i][1]] >= 0)
						moves.push(nxt[x][i][0]);
			if (!moves.length)
				for (let i = 0; i < nxt[x].length; i++)
					moves.push(nxt[x][i][0]);
		}
		console.log("Possible moves: ", moves);
		if (moves.length) move(moves[Math.floor(Math.random() * moves.length)]);
	}
	function toStep(step) {
		stepNumber = step;
		xIsNext = step % 2 === 0;
	}
	function restart() {
		toStep(0);
		computerMove1();
		updateboard();
	}
	function undo() {
		computerMove = -1;
		if (stepNumber > 0) toStep(stepNumber - 1);
		updateboard();
	}
	function redo() {
		computerMove = -1;
		if (stepNumber < history.length - 1) toStep(stepNumber + 1);
		updateboard();
	}
	function setComputerMove(value) {
		computerMove = value;
		computerMove1();
		updateboard();
	}
	function setDifficulty(value) {
		difficulty = value;
		updateboard();
	}

	function updateboard() {
		let current = history[stepNumber];
		let winner = calculateWinner(current.squares);

		let status;
		if (winner) {
			status = "Winner: " + winner;
		} else if (stepNumber >= SZ) {
			status = "Tie";
		} else {
			status = "Next player: " + (xIsNext ? "X" : "O");
		}
		document.getElementById("status").innerHTML = status;

		let board = "";
		for (let i = 0; i < SZ; i++) {
			board += `<div class="tttcell" onClick=handleClick(${i})>${current.squares[i]}</div>`;
		}
		document.getElementById("tttboard").innerHTML = board;
		document.getElementById("computerMove").value = computerMove;
		document.getElementById("difficulty").value = difficulty;

	}
	updateboard();
</script>