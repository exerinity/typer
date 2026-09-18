<script>
  import { onDestroy, onMount } from 'svelte'
  import stats_panel from '../stats_panel.svelte'
  import upload_control from '../upload_control.svelte'
  import typing_panel from '../typing_panel.svelte'
  import { standard_deviation } from '../typing_utils.js'

  const starter_text = `The story so far: In the beginning the Universe was created. This has made a lot of people very angry and been widely regarded as a bad move.`
  let target_text = starter_text
  let typed_text = ''
  let started_at = 0
  let elapsed = 0
  /**
     * @type {number | undefined}
     */
  let interval
  let samples = []
  let file_name = 'universe.txt'
  let locked = false
  let animations_enabled = true
  let animation_event = { id: 0, fast: false, shake_multiplier: 0, duration: 0 }
  let animation_id = 0
  let last_input_at = 0

  $: correct_chars = [...typed_text].filter((char, index) => char === target_text[index]).length
  $: accuracy = typed_text.length ? Math.round((correct_chars / typed_text.length) * 100) : 100
  $: wpm = elapsed > 0 ? Math.round(correct_chars / 5 / (elapsed / 60000)) : 0
  $: consistency = samples.length > 1 ? Math.max(0, Math.round(100 - standard_deviation(samples) * 2.5)) : 100
  $: finished = typed_text.length >= target_text.length

  function start_timer() {
    if (started_at || locked) return
    started_at = Date.now()
    interval = setInterval(() => {
      elapsed = Date.now() - started_at
      if (elapsed > 0 && Math.floor(elapsed / 1000) % 2 === 0) samples = [...samples, wpm].slice(-30)
    }, 100)
  }

  function stop_timer() {
    clearInterval(interval)
    elapsed = started_at ? Date.now() - started_at : 0
  }

  function handle_input(event) {
    if (locked) return
    const now = Date.now()
    const key_gap = last_input_at ? now - last_input_at : 0
    typed_text = event.currentTarget.value
    start_timer()
    if (typed_text.length > target_text.length) typed_text = typed_text.slice(0, target_text.length)
    const current_wpm = elapsed > 0 ? Math.round(correct_chars / 5 / (elapsed / 60000)) : 0
    const pace = key_gap > 0 ? Math.min(2, 180 / key_gap) : 1
    const fast = pace >= 1 || current_wpm >= 45
    const control = Math.min(1, (accuracy + consistency) / 200)
    animation_id += 1
    animation_event = {
      id: animation_id,
      fast,
      shake_multiplier: Math.min(3, 0.6 + pace * 0.8 + control * 0.6),
      duration: fast ? 900 : 650,
    }
    last_input_at = now
    if (typed_text.length >= target_text.length) {
      locked = true
      stop_timer()
    }
  }

  function reset() {
    stop_timer()
    typed_text = ''
    started_at = 0
    elapsed = 0
    samples = []
    locked = false
    last_input_at = 0
    animation_event = { id: 0, fast: false, shake_multiplier: 0, duration: 0 }
  }

  function load_file(event) {
    const file = event.target.files?.[0]
    if (!file) return
    const reader = new FileReader()
    reader.onload = () => {
      target_text = String(reader.result || '').trim() || starter_text
      file_name = file.name
      reset()
    }
    reader.readAsText(file)
  }

  onDestroy(stop_timer)
  onMount(() => {
    animations_enabled = !window.matchMedia('(prefers-reduced-motion: reduce)').matches
  })
</script>

<svelte:head>
  <title>Typing test</title>
</svelte:head>

<main class="app_shell container py-5">
  <section class="intro row align-items-end g-4">
    <div><h1>Typing test</h1></div>
    <div class="controls">
      <label class="animation_toggle"><input type="checkbox" bind:checked={animations_enabled} /> Animations?</label>
      <svelte:component this={upload_control} file_name={file_name} on_load={load_file} />
    </div>
  </section>

  <svelte:component this={stats_panel} {wpm} {accuracy} {consistency} {elapsed} {started_at} {finished} />
  <svelte:component this={typing_panel} {target_text} {typed_text} {locked} {finished} {reset} {handle_input} {animations_enabled} {animation_event} />
</main>
