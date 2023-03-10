<script lang="ts">
  import AddCustomModeModal from './CustomTimerModal.svelte';

  import type { CustomMode } from '../types/event';

  enum State {
    Idle = 'idle',
    InProgress = 'in progress',
    Resting = 'resting',
    ShortResting = 'shortresting',
    LongResting = 'longresting',
    CustomResting = 'customresting',
  }

  const minutesToSeconds = (minutes: number) => minutes * 60;
  const secondsToMinutes = (seconds: number) => Math.floor(seconds / 60);
  const padWithZeroes = (number: number) => number.toString().padStart(2, '0');

  const POMODORO_S = minutesToSeconds(25);
  const LONG_BREAK_S = minutesToSeconds(20);
  const SHORT_BREAK_S = minutesToSeconds(5);
  let   CUSTOM_MODE_S: number;
  
  let currentState = State.Idle;
  let pomodoroTime = POMODORO_S;
  let completedPomodoros = 0;

  let interval: number;

  let showModal = false;
  let customModes: CustomMode[] = [];
  
  function formatTime(timeInSeconds: number) { 
    const minutes = secondsToMinutes(timeInSeconds);
    const remainingSeconds = timeInSeconds % 60;
    return `${padWithZeroes(minutes)}:${padWithZeroes(remainingSeconds)}`;
  }

  function startPomodoro() { 
    setState(State.InProgress);
    interval = setInterval(() => {
      if (pomodoroTime === 0) {
        completePomodoro();
      }
      pomodoroTime -= 1;
    },1000);
  }

  function startShortBreak() { 
    rest(SHORT_BREAK_S);
  }
  
  function startLongBreak(){
    rest(LONG_BREAK_S);
  }

  function startCustomMode(min: number, sec: number) {
    CUSTOM_MODE_S = minutesToSeconds(min) + sec;
    rest(CUSTOM_MODE_S);
  }

  function setState(newState){
    clearInterval(interval)
    currentState = newState;
  }

  function completePomodoro(){
    completedPomodoros++;
    if (completedPomodoros === 4) {
      rest(LONG_BREAK_S);
      completedPomodoros = 0;
    } else {
      rest(SHORT_BREAK_S);
    }
  }
  
  function rest(time: number){
    switch (time) {
      case SHORT_BREAK_S:
        setState(State.ShortResting);
        break;
      case LONG_BREAK_S:
        setState(State.LongResting);
        break;
      default:
        setState(State.CustomResting);
    }
    pomodoroTime = time;
    interval = setInterval(() => {
      if (pomodoroTime === 0) {
        idle();
      }
      else
        pomodoroTime -= 1;
    },1000);
  }

  function cancelPomodoro() {
    idle();
  }

  function idle(){
    switch (currentState) {
      case State.ShortResting:
        pomodoroTime = SHORT_BREAK_S;
        break;
      case State.LongResting:
        pomodoroTime = LONG_BREAK_S;
        break;
      case State.CustomResting:
        pomodoroTime = CUSTOM_MODE_S;
        break;
      default:
        pomodoroTime = POMODORO_S;
    }
    setState(State.Idle);
  }

  function addCustomMode(e) {
    const customMode = e.detail;
    customModes = [...customModes, customMode];
  }

  function deleteCustomMode(id) {
    customModes = customModes.filter((customMode) => customMode.id != id);
    pomodoroTime = POMODORO_S;
  }
</script>
  


<section>
  <div class="prog">  
    <div class="timer">
      <time class="actual-time">
        {formatTime(pomodoroTime)}
      </time>
    </div>
  </div>
    <div class="button-class">
      <button class="time-start" on:click={startPomodoro} disabled={currentState !== State.Idle}>Start</button>
      <button on:click={startShortBreak} disabled={currentState !== State.Idle}>Short Break</button>
      <button on:click={startLongBreak} disabled={currentState !== State.Idle}>Long Break</button>
      <button on:click={cancelPomodoro} disabled={currentState === State.Idle}>Cancel</button>
      {#each customModes as customMode}
        <div class="custom-mode-select">
          <button on:click={() => {
              startCustomMode(customMode.minutes, customMode.seconds);
            }} disabled={currentState !== State.Idle}>
            {customMode.name}
          </button>
          <button class="danger-action" on:click={() => {
              deleteCustomMode(customMode.id);
            }} disabled={currentState !== State.Idle}>
            Delete
          </button>
        </div>
      {/each}
      <button class="fadedtext" on:click={() => (showModal = true)} disabled={currentState !== State.Idle}>+ Custom Mode</button>
      <AddCustomModeModal bind:showModal on:addCustomMode={addCustomMode} />
    </div>
</section>

<style>
  time {
    display: block;
    font-size: 5em;
    margin-bottom: 0.2em;
  }

  .prog {
    border-radius: 50%;
    background: conic-gradient(#ffffff 0deg, rgb(200, 200, 200, 0.4) 0deg);
    width: 300px;
    height: 300px;
    align-items: center;
    text-align: center;
    margin: 25px 0px;
  }

  .timer {
    height: 290px;
    width: 290px;
    border-radius: 50%;
    background-color: var(--evergreen-dark);
    top: 50%;
    position: relative;
    transform: translateY(-50%);
    margin: auto;
  }

  .actual-time {
    font-family: 'Space Grotesk';
    font-weight: bold;
    top: 50%;
    position: relative;
    transform: translateY(-50%);
  }

  button {
    text-align: center;
    color: white;
    font-family: 'Space Grotesk';
    font-size: 20px;
    font-weight: bold;
    width: 180px;
    padding: 7px 0px;
    margin: 20px;
    border-radius: 24px;
    border-width: 3px;
    border-style: solid;
    border-color: white;
    background-color: transparent;
  }

  button:disabled {
    border-color: var(--evergreen-faded);
    color: var(--evergreen-faded);
    background-color: transparent;
  }

  .time-start {
    background-color: var(--orange-light);
    border-color: var(--orange-light);
  
    transition: background 0.25s ease-in-out, border 0.25s ease-in-out;
  }

  .time-start:hover {
    background: var(--orange-faded);
    border-color: var(--orange-faded);
  }

  .danger-action {
    background-color: rgb(231, 67, 67);
    border-color: rgb(231, 67, 67);
  }
  
  .button-class button {
    display: block;
  }

  .custom-mode-select {
    display: inline-flex;
    margin-block-start: -20px;
  }

</style>