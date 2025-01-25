<script lang="ts">
  import { AppShell } from '@skeletonlabs/skeleton';
  import Timer from './lib/Timer.svelte';
</script>

<AppShell class="dark">
  <main class="container mx-auto flex items-center justify-center min-h-screen bg-black">
    <Timer />
  </main>
</AppShell>