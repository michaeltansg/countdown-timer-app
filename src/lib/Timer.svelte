<script lang="ts">
  import { onDestroy } from 'svelte';

  let hours = 0;
  let minutes = 0;
  let seconds = 0;
  let isRunning = false;
  let isPaused = false;
  let animationFrameId: number;
  let progress = 0;
  let initialTotalSeconds = 0;
  let currentTotalSeconds = 0;
  let startTime: number;
  let pausedTimeRemaining = 0;

  $: radius = 120;
  $: circumference = 2 * Math.PI * radius;
  $: strokeDashoffset = circumference * (1 - progress);

  function setAndStartTimer(totalSeconds: number) {
    if (!isRunning && !isPaused) {
      resetTimer();
      hours = Math.floor(totalSeconds / 3600);
      minutes = Math.floor((totalSeconds % 3600) / 60);
      seconds = totalSeconds % 60;
      initialTotalSeconds = totalSeconds;
      currentTotalSeconds = totalSeconds;
      startTimer();
    }
  }

  function updateTimer(timestamp: number) {
    if (!startTime) startTime = timestamp;
    const elapsedTime = (timestamp - startTime) / 1000; // Convert to seconds
    
    if (isRunning) {
      currentTotalSeconds = initialTotalSeconds - elapsedTime;
      
      if (currentTotalSeconds <= 0) {
        currentTotalSeconds = 0;
        progress = 1;
        isRunning = false;
        return;
      }

      progress = 1 - (currentTotalSeconds / initialTotalSeconds);
      
      hours = Math.floor(currentTotalSeconds / 3600);
      minutes = Math.floor((currentTotalSeconds % 3600) / 60);
      seconds = Math.floor(currentTotalSeconds % 60);
      
      animationFrameId = requestAnimationFrame(updateTimer);
    }
  }

  function startTimer() {
    if (!isRunning && (currentTotalSeconds > 0)) {
      isRunning = true;
      isPaused = false;
      
      if (pausedTimeRemaining > 0) {
        currentTotalSeconds = pausedTimeRemaining;
        // Don't update initialTotalSeconds here to maintain the original progress calculation
      }
      
      startTime = performance.now() - ((initialTotalSeconds - currentTotalSeconds) * 1000);
      animationFrameId = requestAnimationFrame(updateTimer);
    }
  }

  function stopTimer() {
    cancelAnimationFrame(animationFrameId);
    isRunning = false;
    isPaused = true;
    pausedTimeRemaining = currentTotalSeconds;
  }

  function resetTimer() {
    cancelAnimationFrame(animationFrameId);
    isRunning = false;
    isPaused = false;
    hours = 0;
    minutes = 0;
    seconds = 0;
    progress = 0;
    initialTotalSeconds = 0;
    currentTotalSeconds = 0;
    pausedTimeRemaining = 0;
    startTime = 0;
  }

  onDestroy(() => {
    cancelAnimationFrame(animationFrameId);
  });
</script>

<div class="card p-8 space-y-6 bg-black text-white">
  <h2 class="h2 text-center">Countdown Timer</h2>
  
  <div class="flex justify-center gap-4 mb-8">
    <button 
      class="btn variant-filled-primary" 
      on:click={() => setAndStartTimer(60)}
      disabled={isRunning || isPaused}
    >
      60s
    </button>
    <button 
      class="btn variant-filled-primary" 
      on:click={() => setAndStartTimer(90)}
      disabled={isRunning || isPaused}
    >
      90s
    </button>
    <button 
      class="btn variant-filled-primary" 
      on:click={() => setAndStartTimer(120)}
      disabled={isRunning || isPaused}
    >
      120s
    </button>
  </div>

  <div class="relative w-[300px] h-[300px] mx-auto">
    <svg class="w-full h-full -rotate-90 transform" viewBox="0 0 300 300">
      <!-- Background circle -->
      <circle
        cx="150"
        cy="150"
        r={radius}
        stroke="currentColor"
        stroke-width="8"
        fill="none"
        class="text-surface-700"
      />
      <!-- Progress circle -->
      <circle
        cx="150"
        cy="150"
        r={radius}
        stroke="#22c55e"
        stroke-width="8"
        fill="none"
        stroke-linecap="butt"
        style="stroke-dasharray: {circumference}; stroke-dashoffset: {strokeDashoffset}"
      />
    </svg>
    
    <!-- Timer display in the middle -->
    <div class="absolute inset-0 flex items-center justify-center">
      <div class="text-4xl font-bold text-white">
        {hours.toString().padStart(2, '0')}:{minutes.toString().padStart(2, '0')}:{seconds.toString().padStart(2, '0')}
      </div>
    </div>
  </div>

  <div class="flex justify-center gap-4 mt-8">
    {#if isRunning}
      <button class="btn variant-filled-warning" on:click={stopTimer}>Pause</button>
    {:else if isPaused}
      <button class="btn variant-filled-success" on:click={startTimer}>Resume</button>
    {/if}
    <button class="btn variant-filled-error" on:click={resetTimer}>Reset</button>
  </div>
</div>