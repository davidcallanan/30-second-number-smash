<script>

	import { onMount } from "svelte";

	let mode = "classic";
	let state = "start";
	let subState;
	let score;
	let highScore = 0;
	let timeToFinish;
	let timeLeftInterval;
	let timeLeft;
	let question;
	let answers;
	let answerCorrect;
	let startGameReady = true;
	let isGreen;
	let greenTimeout;
	let isRed;
	let redTimeout;
	let stunAmt;
	let stunTime;
	let stunTimeout;
	let stunCount;
	let stunType;
	let endGameTimeout;

	onMount(() => {
		highScore = parseInt(window.localStorage.getItem("tsns-highScore")) || 0;
	});

	const performUpAction = () => {
		if (state === "start" && startGameReady) {
			startGame();
		} else if (state === "playing") {
			if (subState === "question" && answers[0]) {
				guess(0);
			} else if (subState === "stun") {
				resetStun();
			}
		}
	};

	const performLeftAction = () => {
		if (state === "playing") {
			if (subState === "question" && answers[1]) {
				guess(1);
			} else if (subState === "stun") {
				resetStun();
			}
		}
	};

	const performRightAction = () => {
		if (state === "playing") {
			if (subState === "question" && answers[2]) {
				guess(2);
			} else if (subState === "stun") {
				resetStun();
			}
		}
	};

	const performDownAction = () => {
		if (state === "playing") {
			if (subState === "question" && answers[3]) {
				guess(3);
			} else if (subState === "stun") {
				resetStun();
			}
		}
	};

	const startGame = () => {
		startGameReady = false;
		state = "playing";
		score = 0;
		timeToFinish = Date.now() + 30000;
		timeLeft = "30.0";
		stunCount = 0;
		if (stunTimeout) {
			clearTimeout(stunTimeout);
			stunTimeout = undefined;
		}
		endGameTimeout = setTimeout(endGame, 30000);

		if (timeLeftInterval) {
			clearInterval(timeLeftInterval);
		}

		timeLeftInterval = setInterval(updateTimeLeft, 100);

		loadNextQuestion();
	};

	const randint = (min, max) => {
		return Math.floor(Math.random() * (max - min + 1)) + min;
	};

	const zeroPad = (num, places) => String(num).padStart(places, "0");

	const stringCombinations = (str) => {
		let results = [];

		if (str.length === 1) {
			results.push(str);
			return results;
		}

		for (let i = 0; i < str.length; i++) {
			let firstChar = str[i];
			let charsLeft = str.substring(0, i) + str.substring(i + 1);
			let innerPermutations = stringCombinations(charsLeft);
			for (let j = 0; j < innerPermutations.length; j++) {
				results.push(firstChar + innerPermutations[j]);
			}
		}

		return results;
	};

	const uniqueStringCombinations = (str) => {
		return [...new Set(stringCombinations(str))];
	};

	const shuffleArray = (array) => {
		for (let i = array.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		}
	};

	const randArray = (arr) => {
		return arr[Math.floor(Math.random() * arr.length)];
	};

	const range = (start, end) => {
		return Array(end - start + 1).fill().map((_, idx) => start + idx);
	};

	const randomQuestion = () => {
		// @ts-ignore
		return ({
			classic() {
				if (score < 1) {
					// ADD LEVEL 1
					let firstNumber = randint(10, 50);
					let secondNumber = randint(10, Math.min(50, 99 - firstNumber));
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} + ${secondNumberString}`;
					let correctAnswer = firstNumber + secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 2) {
					// SUB LEVEL 1
					let firstNumber = randint(20, 99);
					let secondNumber = randint(10, firstNumber);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} - ${secondNumberString}`;
					let correctAnswer = firstNumber - secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 3) {
					// ADD LEVEL 2
					let firstNumber = randint(10, 99);
					let secondNumber = randint(Math.max(10, 100 - firstNumber), 99);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} + ${secondNumberString}`;
					let correctAnswer = firstNumber + secondNumber;
					return zeroPad(correctAnswer, 3);
				} else if (score < 4) {
					// SUB LEVEL 2
					let firstNumber = randint(20, 198);
					let secondNumber = randint(10, firstNumber);
					let firstNumberString = zeroPad(firstNumber, 3);
					let secondNumberString = zeroPad(secondNumber, 3);
					question = `${firstNumberString} - ${secondNumberString}`;
					let correctAnswer = firstNumber - secondNumber;
					return zeroPad(correctAnswer, 3);
				} else if (score < 5) {
					// MULT LEVEL 1
					let firstNumber = randint(1, 14);
					let secondNumber = randint(1, 14);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} * ${secondNumberString}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 3);
				} else if (score < 6) {
					// MULT LEVEL 2
					let firstNumber = randint(4, 31);
					let secondNumber = randint(4, 31);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} * ${secondNumberString}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 3);
				} else if (score < 7) {
					// RANDOM BASES QUESTION
					let choice = randArray(["dec-to-bin", "dec-to-hex", "bin-to-dec", "hex-to-dec", "bin-to-hex", "hex-to-bin"]);

					if (choice === "dec-to-bin") {
						let number = randint(0, 15);
						question = `Convert ${number} to binary`;
						let correctAnswer = number.toString(2);
						return zeroPad(correctAnswer, 4);
					} else if (choice === "dec-to-hex") {
						let number = randint(0, 255);
						question = `Convert ${number} to hex`;
						let correctAnswer = number.toString(16).toUpperCase();
						return zeroPad(correctAnswer, 2);
					} else if (choice === "bin-to-dec") {
						let number = randint(0, 63);
						let numberBin = zeroPad(number.toString(2), 6);
						question = `Convert ${numberBin} to decimal`;
						return zeroPad(number, 3);
					} else if (choice === "hex-to-dec") {
						let number = randint(0, 255);
						let numberHex = zeroPad(number.toString(16), 2).toUpperCase();
						question = `Convert hex ${numberHex} to decimal`;
						return zeroPad(number, 3);
					} else if (choice === "bin-to-hex") {
						let number = randint(0, 255);
						let numberBin = zeroPad(number.toString(2), 8).toUpperCase();
						let numberHex = zeroPad(number.toString(16), 2).toUpperCase();
						question = `Convert ${numberBin} to hex`;
						return numberHex;
					} else if (choice === "hex-to-bin") {
						let number = randint(0, 15);
						let numberHex = zeroPad(number.toString(16), 1).toUpperCase();
						let numberBin = zeroPad(number.toString(2), 4).toUpperCase();
						question = `Convert hex ${numberHex} to binary`;
						return numberBin;
					}
				} else if (score < 8) {
					// MULT LEVEL 3
					let firstNumber = randint(10, 50);
					let secondNumber = randint(10, 50);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} * ${secondNumberString}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 4);
				} else if (score < 9) {
					// MULT LEVEL 4
					let firstNumber = randint(10, 70);
					let secondNumber = randint(10, 70);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} * ${secondNumberString}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 4);
				} else {
					// MULT LEVEL 5
					let firstNumber = randint(10, 99);
					let secondNumber = randint(10, 99);
					let firstNumberString = zeroPad(firstNumber, 2);
					let secondNumberString = zeroPad(secondNumber, 2);
					question = `${firstNumberString} * ${secondNumberString}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 4);
				}
			},
			tables_12() {
				let firstNumber = randint(1, 12);
				let secondNumber = randint(1, 12);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			tables_16() {
				let firstNumber = randint(2, 16);
				let secondNumber = randint(2, 16);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			tables_20() {
				let firstNumber = randint(2, 20);
				let secondNumber = randint(2, 20);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			tables_24() {
				let firstNumber = randint(2, 24);
				let secondNumber = randint(2, 24);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			tables_28() {
				let firstNumber = randint(2, 28);
				let secondNumber = randint(2, 28);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			tables_32() {
				let firstNumber = randint(2, 32);
				let secondNumber = randint(2, 32);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			multiplication_easy() {
				let firstNumber = randint(10, Math.min(13 + score, 31));
				let secondNumber = randint(2, Math.min(13 + score, 31));
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			multiplication_hard() {
				if (score < 2) {
					let firstNumber = randint(1, 18);
					let secondNumber = randint(1, 18);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 4) {
					let firstNumber = randint(1, 24);
					let secondNumber = randint(1, 24);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 6) {
					let firstNumber = randint(10, 31);
					let secondNumber = randint(2, 31);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 7) {
					let firstNumber = randArray([...range(10, 31), 35, 40, 45, 50]);
					let secondNumber = randArray([...range(2, 31), 35, 40, 45, 50]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 8) {
					let firstNumber = randArray([...range(10, 31), 35, 40, 45, 50, 55, 60, 65, 70]);
					let secondNumber = randArray([...range(2, 31), 35, 40, 45, 50, 55, 60, 65, 70]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 9) {
					let firstNumber = randArray([...range(10, 31), 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					let secondNumber = randArray([...range(2, 31), 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 10) {
					let firstNumber = randArray([...range(10, 31), 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					let secondNumber = randArray([...range(2, 31), 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 11) {
					let firstNumber = randArray([...range(10, 50), 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					let secondNumber = randArray([...range(2, 50), 55, 60, 65, 70, 75, 80, 85, 90, 95]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 12) {
					let firstNumber = randArray([...range(10, 70), 70, 75, 80, 85, 90, 95]);
					let secondNumber = randArray([...range(2, 70), 70, 75, 80, 85, 90, 95]);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				} else if (score < 13) {
					let firstNumber = randint(10, 99);
					let secondNumber = randint(2, 99);
					question = `${firstNumber} * ${secondNumber}`;
					let correctAnswer = firstNumber * secondNumber;
					return zeroPad(correctAnswer, 2);
				}
			},
			mult_easy() {
				let x;
				if (score < 8) {
					x = 6;
				} else if (score < 16) {
					x = 12;
				} else {
					x = 24;
				}
				let firstNumber = randint(1, x) * 4;
				let secondNumber = randint(1, x) * 4;
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			mult_med() {
				let firstNumber, secondNumber;
				if (score < 6) {
					firstNumber = randint(1, 12) * 2;
					secondNumber = randint(1, 12) * 2;
				} else if (score < 12) {
					firstNumber = randint(1, 24) * 2;
					secondNumber = randint(1, 24) * 2;
				} else {
					firstNumber = randint(1, 49) * 2;
					secondNumber = randint(1, 49) * 2;
				}
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			mult_hard() {
				let firstNumber, secondNumber;
				if (score < 4) {
					firstNumber = randint(1, 24);
					secondNumber = randint(1, 24);
				} else if (score < 8) {
					firstNumber = randint(1, 49);
					secondNumber = randint(1, 49);
				} else {
					firstNumber = randint(1, 99);
					secondNumber = randint(1, 99);
				}
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			near_squares_easy() {
				let spread = Math.floor(score / 4);
				let firstNumber = randint(1, 32);
				let secondNumber = firstNumber - randint(0, spread);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
			near_squares_hard() {
				let spread = Math.floor(score / 4);
				let firstNumber = randint(1, 99);
				let secondNumber = firstNumber - randint(0, spread);
				question = `${firstNumber} * ${secondNumber}`;
				let correctAnswer = firstNumber * secondNumber;
				return zeroPad(correctAnswer, 2);
			},
		})[mode]();
	};

	const loadNextQuestion = () => {
		subState = "question";

		let correctAnswerString = randomQuestion();

		let answerCombinations = uniqueStringCombinations(correctAnswerString);

		if (answerCombinations.length < 2) {
			loadNextQuestion();
			return;
		}

		answerCombinations.splice(answerCombinations.indexOf(correctAnswerString), 1);
		shuffleArray(answerCombinations);
		answerCombinations = answerCombinations.slice(0, 3);
		answerCombinations.push(correctAnswerString);
		shuffleArray(answerCombinations);

		if (answerCombinations.length == 2) {
			answerCombinations = [undefined, answerCombinations[0], answerCombinations[1]];
		}

		answerCorrect = correctAnswerString;
		answers = answerCombinations;
	};

	const updateTimeLeft = () => {
		timeLeft = ((timeToFinish - Date.now()) / 1000).toFixed(1);
	};

	const endGame = () => {
		redFlash();

		if (score > highScore) {
			highScore = score;
			window.localStorage.setItem("tsns-highScore", highScore);
		}

		state = "start";

		setTimeout(() => {
			startGameReady = true;
		}, 1000);
	};

	const handleKeyDown = (event) => {
		if (event.key === "ArrowLeft") {
			performLeftAction();
		} else if (event.key === "ArrowRight") {
			performRightAction();
		} else if (event.key === "ArrowUp") {
			performUpAction();
		} else if (event.key === "ArrowDown") {
			performDownAction();
		} else if (event.key === "s") {
			if (state === "playing" && subState === "question") {
				skip();
			}
		}
	};

	const skip = () => {
		guess(answers.indexOf(answerCorrect));
	};

	const greenFlash = () => {
		isGreen = true;
		greenTimeout && clearTimeout(greenTimeout);
		greenTimeout = setTimeout(() => {
			isGreen = false;
		}, 100);
	};

	const redFlash = () => {
		isRed = true;
		redTimeout && clearTimeout(redTimeout);
		redTimeout = setTimeout(() => {
			isRed = false;
		}, 1000);
	};

	const guess = (idx) => {
		console.log("Guessing", idx);

		if (answers[idx] === answerCorrect) {
			console.log("Correct!");
			score++;
			greenFlash();
			loadNextQuestion();
			timeToFinish += 500;
			clearTimeout(endGameTimeout);
			endGameTimeout = setTimeout(endGame, timeToFinish - Date.now());
		} else {
			console.log("Incorrect - Preparing to stun!");
			// sort answers by distance from correct answer

			let correctAnswer = Number(answerCorrect);

			let sortedAnswers = answers.filter((a) => a !== undefined).sort((a, b) => {
				let aDiff = Math.abs(Number(a) - correctAnswer);
				let bDiff = Math.abs(Number(b) - correctAnswer);
				return aDiff - bDiff;
			});

			let distIdx = sortedAnswers.indexOf(answers[idx]);

			stun(distIdx - 1);
		}
	};

	const stun = (amt) => {
		console.log("Stun!", amt);

		subState = "stun";
		stunAmt = amt;
		stunCount++;
		stunType = stunAmt === 0 ? "short" : stunAmt === 1 ? "medium" : "long";

		stunTime = stunAmt === 0 ? 200 : stunAmt === 1 ? 800 : 1600;
		stunTime *= Math.pow(2, Math.min(stunCount, 6) - 1);

		stunTimeout = setTimeout(() => {
			loadNextQuestion();
		}, stunTime);
	};

	const resetStun = () => {
		stunTimeout && clearTimeout(stunTimeout);
		
		stunTimeout = setTimeout(() => {
			loadNextQuestion();
		}, stunTime);
	};

</script>

<svelte:window on:keydown={handleKeyDown} />

<div class="wrapper" style="display: flex; flex-direction: column; justify-content: space-evenly;">
	<div style="text-align: center; display: flex; flex-direction: column; align-items: center;">
		<h3 style="font-weight: 600;" class="title"> 30 Second Number Smash </h3>
		<h4 style="font-weight: 400; padding-top: 0.25rem; color: #555;"> Tip: use arrow keys on a keyboard </h4>
		<div style="margin-top: 0.25rem;">
			<span style="font-size: 0.75rem; font-weight: 600;">Mode:</span>
			<select bind:value={mode}>
				<option value="classic">Classic</option>
				<option value="tables_12">Tables (12)</option>
				<option value="tables_16">Tables (16)</option>
				<option value="tables_20">Tables (20)</option>
				<option value="tables_24">Tables (24)</option>
				<option value="tables_28">Tables (28)</option>
				<option value="tables_32">Tables (32)</option>
				<option value="mult_easy">Mult (Easy)</option>
				<option value="mult_med">Mult (Medium)</option>
				<option value="mult_hard">Mult (Hard)</option>
				<option value="near_squares_easy">Near Squares (Easy)</option>
				<option value="near_squares_hard">Near Squares (Hard)</option>
			</select>
		</div>
	</div>
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<div class="main" class:green={isGreen} class:red={isRed}
		class:stun-short={state === "playing" && subState === "stun" && stunType === "short"}
		class:stun-medium={state === "playing" && subState === "stun" && stunType === "medium"}
		class:stun-long={state === "playing" && subState === "stun" && stunType === "long"}
		on:click={() => {
			if (state === "playing" && subState === "stun") {
				resetStun();
			}
		}}
	>
		{#if state === "start"}
			<div>
				{#if score !== undefined}
					<h3> You scored {score}! </h3>
				{/if}
			</div>
			{#if startGameReady}
				<button style="letter-spacing: 0.0625rem;" on:click={startGame}> Start New Game </button>
			{:else}
				<div>
					<div>
						<b>GAME OVER</b><br>You ran out of time!
					</div>
				</div>
			{/if}
			<div>
				{#if highScore}
					High score: {highScore}
				{/if}
			</div>
			<div>
			</div>
			<div>
				Welcome to 30 Second Number Smash!
			</div>
			<div/>
			<div/>
			<div>
			</div>
		{:else if state === "playing"}
			<div> Score: {score} </div>
			{#if subState === "question" && answers[0]}
				<button class="answer" on:click={() => guess(0)}>
					{answers[0]}
				</button>
			{:else if subState === "stun"}
				<p> INCORRECT </p>
			{:else}
				<div/>
			{/if}
			<div>
				Time left: {timeLeft}s
			</div>
			{#if subState === "question" && answers[1]}
				<button class="answer" on:click={() => guess(1)}>
					{answers[1]}
				</button>
			{:else}
				<div/>
			{/if}
			<div>
				{#if subState === "question"}
					<b style="font-size: 1.25rem;">{question}</b>
				{:else if subState === "stun"}
					<p style="font-size: 1.25rem; font-weight: 600;">
						{#if stunType === "short"}
							Short stun!
						{:else if stunType === "medium"}
							Medium stun!
						{:else if stunType === "long"}
							Long stun!
						{/if}
					</p>
				{/if}
			</div>
			{#if subState === "question" && answers[2]}
				<button class="answer" on:click={() => guess(2)}>
					{answers[2]}
				</button>
			{:else}
				<div/>
			{/if}
			<div/>
			{#if subState === "question" && answers[3]}
				<button class="answer" on:click={() => guess(3)}>
					{answers[3]}
				</button>
			{:else if subState === "stun"}
				<p> Stun time: {(stunTime / 1000).toFixed(1)}s </p>
			{:else}
				<div/>
			{/if}
		{/if}
	</div>
</div>

<div class="footer">
	<p>30 Second Number Smash <small><i>by David P. Callanan</i></small> <span style="width: 0.25rem; display: inline-block;"></span> <span class="version">(version 0.1.1)</span></p>
	<a href="https://github.com/davidcallanan/30-second-number-smash" target="_blank"><div class="vog">
		View on GitHub
	</div></a>
</div>

<style>

	.wrapper {
		display: flex;
		flex-direction: column;
		height: 100%;
		max-width: 75vh;
		margin: 0 auto;
		outline: 2px solid #bbb;
		background: rgb(246, 246, 246);
	}

	.main {
		display: grid;
		aspect-ratio: 1;
		grid-template-columns: 1fr 1fr 1fr;
		grid-template-rows: 1fr 1fr 1fr;
		outline: 2px solid #bbb;
		background: white;
	}

	.main > * {
		display: flex;
		justify-content: center;
		align-items: center;
		text-align: center;
	}

	.main button.answer {
		font-size: 1.25rem;
		letter-spacing: 0.0625rem;
	}

	.main.green {
		background: rgb(77, 209, 48);
		background: rgb(135, 204, 120);
	}
	
	.main.red {
		background: rgb(204, 120, 120);
	}

	.main:is(.green, .red, .stun-short, .stun-medium, .stun-long) button {
		background: transparent;
	}

	.stun-short {
		background: rgb(204, 166, 120);
	}

	.stun-medium {
		background: rgb(142, 120, 204);
	}

	.stun-long {
		background: rgb(93, 54, 54);
		color: white;
	}

	.footer {
		font-family: Tahoma;
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		border-top: 1px solid #ddd;
		background: #eee;
		display: flex;
		padding: 0.25rem;
		align-items: center;
		justify-content: center;
		font-size: 0.875rem;
	}

	.footer .version {
		color: gray;
	}

	.dark .footer .version {
		color: lightgray;
	}

	.footer p {
		margin: 0;
		margin-right: 2rem;
	}

	.vog {
		background: white;
		border-radius: 1rem;
		padding: 0.25rem 0.75rem;
	}

	.footer a {
		color: #2196F3 !important;
		text-decoration: none;
	}

	@media (max-width: 500px) {
		.footer {
			font-size: 0.675rem;
		}
	}

	.title {
		color: #7932a8;
		animation: colorrr 1.5s infinite alternate;
	}
	
	@keyframes colorrr {
		0% {
			color: #7932a8;
			/* background: white; */
		}
		
		50% {
			color: #3c7851;
			/* background: rgb(255, 255, 210); */
		}
		
		100% {
			color: #a83279;
			/* background: white; */
		}
	}

</style>
