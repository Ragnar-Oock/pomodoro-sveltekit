<script lang="ts">
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
		 */
		next: () => void;
	}

	const unstartedState = (): PomodoroState =>({
		name: "unstarted",
		duration: 0,
		next: () => timer.state = workingState()
	});
	const workingState = (count: number = 0): PomodoroState =>({
		name: "working",
		duration: 20 * 60,
		next: () => {
			if (count > 3) {
				timer.state = longBreakState();
			}
			else {
				timer.state = shortBreakState(count);
			}
		}
	});
	const shortBreakState = (count: number = 0): PomodoroState =>({
		name: "short break",
		duration: 5 * 60,
		next: () => timer.state = workingState(count++),
	});
	const longBreakState = (): PomodoroState =>({
		name: "long break",
		duration: 30 * 60,
		next: () => {
			timer.state = unstartedState();
		},
	});

	let timer = $state<{state: PomodoroState}>({
		state: unstartedState(),
	});

	let remaining = $state(0);

	const minutes = $derived.by(() => Math.floor(remaining / 60));
	const seconds = $derived.by(() => Math.floor(remaining - (minutes * 60)));

	// state = unstarted
	// state = working for x minutes
	// state = short break for y minutes
	// state = long break for z minutes

	let startingTime: DOMHighResTimeStamp | undefined = undefined;

	let rAF;
	function start() {
		const callback = timeStamp => {
			startingTime ??= timeStamp;

			// how long have we ran in seconds
			let delta = (timeStamp - startingTime) / 10;

			remaining = timer.state.duration - delta;

			if (remaining < 0) {
				timer.state.next()
			}

			rAF = requestAnimationFrame(callback);
		}

		timer.state = workingState();
		rAF = requestAnimationFrame(callback);
	}

	function pause() {
		cancelAnimationFrame(rAF);
	}

	function doBreak() {
		timer.state = shortBreakState();
	}




</script>

<h1>pomodoro</h1>

<p>{timer.state.name}</p>

<button onclick="{start}">start</button>
<button onclick="{pause}">pause</button>
<button onclick="{doBreak}">break</button>
<span>{minutes}:{seconds}</span>