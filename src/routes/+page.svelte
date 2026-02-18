<script lang="ts">
	let errorMessage = $state('');
	let firstNum = $state('0');
	let secondNum = $state('0');
	let isFirst = $state(true);
	let operator = $state('');
	let activeOperator = $state('');

	const numClick = (s: string) => () => {
		if (isFirst) {
			if (firstNum !== '0') firstNum += s;
			else firstNum = s;
		} else {
			if (secondNum !== '0') secondNum += s;
			else secondNum = s;
		}
	};

	const clear = () => {
		firstNum = '0';
		secondNum = '0';
		operator = '';
		isFirst = true;
		activeOperator = '';
	};

	const back = () => {
		if (isFirst) {
			firstNum = firstNum.slice(0, -1);
			if (firstNum === '') firstNum = '0';
		} else {
			secondNum = secondNum.slice(0, -1);
			if (secondNum === '') secondNum = '0';
		}
	};

	const dotClick = () => {
		if (isFirst && !firstNum.includes('.')) {
			firstNum += '.';
		} else if (!isFirst && !secondNum.includes('.')) {
			secondNum += '.';
		}
	};

	const percentClick = () => {
		if (isFirst) {
			firstNum = (Number(firstNum) / 100).toString();
		} else {
			secondNum = (Number(secondNum) / 100).toString();
		}
	};

	const toggleSign = () => {
		if (isFirst) {
			if (firstNum === '0') return;
			if (firstNum.startsWith('-')) {
				firstNum = firstNum.slice(1);
			} else {
				firstNum = '-' + firstNum;
			}
		} else {
			if (secondNum === '0') return;
			if (secondNum.startsWith('-')) {
				secondNum = secondNum.slice(1);
			} else {
				secondNum = '-' + secondNum;
			}
		}
	};

	const operatorClick = (s: string) => () => {
		if (!isFirst) {
			firstNum = calculate(firstNum, secondNum, operator).toString();
			secondNum = '0';
		}
		isFirst = false;
		operator = s;
		activeOperator = s;
	};

	const equalClick = () => {
		if (isFirst) {
		} else {
			firstNum = calculate(firstNum, secondNum, operator).toString();
			secondNum = '0';
		}
		isFirst = true;
		operator = '';
		activeOperator = '';
	};

	const calculate = (n1: string, n2: string, op: string): number => {
		const round = 1_000_000_000;
		if (op === '÷') {
			if (Number(n2) === 0) {
				errorMessage = 'Cannot divide by zero';
				setTimeout(() => {
					errorMessage = '';
				}, 5000);
				return 0;
			}
			return Math.floor((Number(n1) / Number(n2)) * round) / round;
		}
		if (op === '×') return Math.floor(Number(n1) * Number(n2) * round) / round;
		if (op === '+') return Math.floor((Number(n1) + Number(n2)) * round) / round;
		if (op === '-') return Math.floor((Number(n1) - Number(n2)) * round) / round;
		return 0;
	};

	const formatDisplay = (num: string): string => {
		// Handle very long numbers
		if (num.length > 12) {
			const n = Number(num);
			if (Math.abs(n) >= 1e12 || (Math.abs(n) < 1e-6 && n !== 0)) {
				return n.toExponential(6);
			}
			return num.slice(0, 12);
		}
		return num;
	};

	const handleKeydown = (e: KeyboardEvent) => {
		// Prevent default for calculator keys
		if (
			[
				'0',
				'1',
				'2',
				'3',
				'4',
				'5',
				'6',
				'7',
				'8',
				'9',
				'.',
				'+',
				'-',
				'*',
				'/',
				'x',
				'X',
				'%',
				'Enter',
				'=',
				'Escape',
				'Backspace',
				'F9'
			].includes(e.key)
		) {
			e.preventDefault();
		}

		// Numbers
		if (e.key >= '0' && e.key <= '9') {
			numClick(e.key)();
		}
		// Operators
		else if (e.key === '+') {
			operatorClick('+')();
		} else if (e.key === '-') {
			operatorClick('-')();
		} else if (e.key === '*' || e.key === 'x' || e.key === 'X') {
			operatorClick('×')();
		} else if (e.key === '/') {
			operatorClick('÷')();
		}
		// Special keys
		else if (e.key === 'Enter' || e.key === '=') {
			equalClick();
		} else if (e.key === 'Escape') {
			clear();
		} else if (e.key === 'Backspace') {
			back();
		} else if (e.key === '.') {
			dotClick();
		} else if (e.key === '%') {
			percentClick();
		} else if (e.key === 'F9') {
			toggleSign();
		}
	};

	$effect(() => {
		window.addEventListener('keydown', handleKeydown);
		return () => window.removeEventListener('keydown', handleKeydown);
	});
</script>

<div class="min-h-screen flex-row items-center justify-between gap-8 bg-zinc-950 p-4">
	<header class="p-4 text-center text-4xl font-bold text-white">
		<h1 class="">Calculator</h1>
	</header>

	<div
		class="mx-auto grid max-w-sm flex-1 grid-cols-4 gap-3 rounded-2xl bg-zinc-900 p-4 shadow-2xl"
	>
		<!-- Screen -->
		<div class="col-span-4 space-y-2 rounded-xl bg-black p-4 text-right">
			<p class="min-h-5 text-sm text-zinc-400">
				{#if !isFirst}
					{firstNum + operator}
				{/if}
			</p>
			<p class="text-3xl font-semibold break-all text-white">
				{#if isFirst}
					{formatDisplay(firstNum)}
				{:else}
					{formatDisplay(secondNum)}
				{/if}
			</p>
		</div>

		<!-- First row -->
		<button onclick={clear} class="btn-func">AC</button>
		<button onclick={toggleSign} class="btn-func">+/-</button>
		<button onclick={percentClick} class="btn-func">%</button>
		<button onclick={operatorClick('÷')} class="btn-operator" class:active={activeOperator === '÷'}
			>÷</button
		>

		<!-- Second row -->
		<button onclick={numClick('7')} class="btn-number">7</button>
		<button onclick={numClick('8')} class="btn-number">8</button>
		<button onclick={numClick('9')} class="btn-number">9</button>
		<button onclick={operatorClick('×')} class="btn-operator" class:active={activeOperator === '×'}
			>×</button
		>

		<!-- Third row -->
		<button onclick={numClick('4')} class="btn-number">4</button>
		<button onclick={numClick('5')} class="btn-number">5</button>
		<button onclick={numClick('6')} class="btn-number">6</button>
		<button onclick={operatorClick('+')} class="btn-operator" class:active={activeOperator === '+'}
			>+</button
		>

		<!-- Fourth row -->
		<button onclick={numClick('1')} class="btn-number">1</button>
		<button onclick={numClick('2')} class="btn-number">2</button>
		<button onclick={numClick('3')} class="btn-number">3</button>
		<button onclick={operatorClick('-')} class="btn-operator" class:active={activeOperator === '-'}
			>-</button
		>

		<!-- Fifth row -->
		<button onclick={numClick('0')} class="btn-number col-span-2">0</button>
		<button onclick={dotClick} class="btn-number">.</button>
		<button onclick={equalClick} class="btn-equal">=</button>
	</div>

	<div>
		<p class="text-center text-2xl font-bold text-red-500">{errorMessage}</p>
	</div>
</div>
