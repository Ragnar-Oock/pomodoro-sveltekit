<script lang="ts">
	export {};
	// eslint-disable func-style
	// eslint-disable no-magic-numbers
	type PomodoroState = {
		/**
		 * the display name of the current timer
		 */
		name: string;
		/**
		 * how long the timer needs to run for
		 */
		duration: number;
		/**
		 * update the timer to the next state in the list
		 *
		 * @returns should the timer keep on ticking ?
		 */
		next: () => boolean;
	}

	const unstartedState = (): PomodoroState => ({
		name: "unstarted",
		duration: 0,
		next: (): boolean => {
			timer.state = workingState();
			return true;
		}
	});
	const workingState = (count = 0): PomodoroState => ({
		name: "working",
		duration: 5,
		next: (): boolean => {
			timer.state = count > 3 ? longBreakState() : shortBreakState(count);
			return true;
		}
	});
	const shortBreakState = (count = 0): PomodoroState => ({
		name: "short break",
		duration: 2,
		next: (): boolean => {
			timer.state = workingState(count + 1);
			return true;
		},
	});
	const longBreakState = (): PomodoroState => ({
		name: "long break",
		duration: 10,
		next: (): boolean => {
			timer.state = unstartedState();
			return false;
		},
	});

	let timer = $state<{ state: PomodoroState }>({
		state: unstartedState(),
	});

	// how long do we have until the end of the current counter (in seconds)
	let remaining = $state(0);

	const minutes = $derived.by(() => Math.floor(remaining / 60));
	const seconds = $derived.by(() => Math.floor(remaining - (minutes * 60)));

	// state = unstarted
	// state = working for x minutes
	// state = short break for y minutes
	// state = long break for z minutes

	let startingTime: DOMHighResTimeStamp | undefined;

	let rAF: number | undefined;

	const timeFactor = 10e2; // change the speed of time, the bigger the number, the slower the time

	function loop(timeStamp: DOMHighResTimeStamp): void {
		startingTime ??= timeStamp;

		// how long have we ran in seconds
		let delta = (timeStamp - startingTime) / timeFactor;

		remaining = Math.round(timer.state.duration - delta);

		const state = timer.state.name;
		let keepTicking: boolean;
		if (remaining === 0) {
			keepTicking = timer.state.next();
			startingTime = timeStamp;
		}
		else {
			keepTicking = true;
		}
		if (state !== timer.state.name) {
			console.log(`${ state } => ${timer.state.name} (${timer.state.duration})`)
		}

		if (keepTicking) {
			rAF = requestAnimationFrame(loop);
		}
	}

	function start(): void {
		if (timer.state.name === 'unstarted') {
			timer.state = workingState();
			rAF ??= requestAnimationFrame(loop);
		}
	}

	function stop(): void {
		if (rAF) {
			console.log('stop')
			cancelAnimationFrame(rAF);
			rAF = undefined;
			timer.state = unstartedState();
		}
	}

	// function doBreak(): void {
	// 	timer.state = shortBreakState();
	// }


</script>

<h1>pomodoro</h1>

<p>{timer.state.name}</p>

<button onclick="{start}">start</button>
<button onclick="{stop}">pause</button>
<!--<button onclick="{doBreak}">break</button>-->
<span>{minutes}:{seconds}</span>