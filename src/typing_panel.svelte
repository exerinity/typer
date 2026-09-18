<script>
  import { tick } from 'svelte'

  export let target_text
  export let typed_text
  export let locked
  export let finished
  export let reset
  export let handle_input
  export let animations_enabled
  export let animation_event

  let text_parts = [{ value: '', start: 0, is_space: false }]
  let typing_wrap
  let particles = []

  $: {
    let start = 0
    text_parts = target_text.split(/(\s+)/).map((value) => {
      const part = { value, start, is_space: /^\s+$/.test(value) }
      start += value.length
      return part
    })
  }

  $: if (animations_enabled && animation_event.id) create_burst(animation_event)

  async function create_burst(event) {
    await tick()
    const typed_letters = typing_wrap?.querySelectorAll('.text_display .typed')
    const current = typed_letters?.[typed_letters.length - 1]
    if (!current) return
    const wrap_box = typing_wrap.getBoundingClientRect()
    const letter_box = current.getBoundingClientRect()
    const count = event.fast ? 24 : 12
    const burst = Array.from({ length: count }, (_, index) => ({
      id: `${event.id}-${index}`,
      x: letter_box.left - wrap_box.left + letter_box.width / 2,
      y: letter_box.top - wrap_box.top + letter_box.height / 2,
      angle: Math.random() * 360,
      distance: 30 + Math.random() * (event.fast ? 90 : 50),
      x_offset: 0,
      y_offset: 0,
      duration: event.duration,
      color: ['#ff3b30', '#ffd166', '#ffffff', '#4dabf7'][index % 4],
      size: 3 + Math.random() * 4,
    })).map((particle) => ({
      ...particle,
      x_offset: Math.cos(particle.angle * Math.PI / 180) * particle.distance,
      y_offset: Math.sin(particle.angle * Math.PI / 180) * particle.distance + 24,
    }))
    particles = [...particles, ...burst]
    const panel = typing_wrap.closest('.practice_panel')
    panel?.animate([
      { transform: 'translate3d(0, 0, 0)' },
      { transform: `translate3d(${-3 * event.shake_multiplier}px, 0, 0)` },
      { transform: `translate3d(${3 * event.shake_multiplier}px, 0, 0)` },
      { transform: `translate3d(${-2 * event.shake_multiplier}px, 0, 0)` },
      { transform: `translate3d(${2 * event.shake_multiplier}px, 0, 0)` },
      { transform: 'translate3d(0, 0, 0)' },
    ], { duration: event.duration, easing: 'ease-out' })
    setTimeout(() => {
      particles = particles.filter((particle) => !particle.id.startsWith(`${event.id}-`))
    }, event.duration)
  }
</script>

<section class="practice_panel">
  <div class="panel_heading"><span>Passage / {target_text.length} chars</span></div>
  <div class="typing_wrap" class:complete={finished} bind:this={typing_wrap}>
    {#if animations_enabled}
      <div class="confetti_layer" aria-hidden="true">
        {#each particles as particle (particle.id)}
          <span class="confetti" style={`--x: ${particle.x}px; --y: ${particle.y}px; --angle: ${particle.angle}deg; --x-offset: ${particle.x_offset}px; --y-offset: ${particle.y_offset}px; --color: ${particle.color}; --size: ${particle.size}px; --duration: ${particle.duration}ms`}></span>
        {/each}
      </div>
    {/if}
    <div class="text_display" aria-hidden="true">
      {#each text_parts as part}
        {#if part.is_space}
          {#each [...part.value] as character, index}<span class:typed={part.start + index < typed_text.length} class:correct={part.start + index < typed_text.length && typed_text[part.start + index] === character} class:incorrect={part.start + index < typed_text.length && typed_text[part.start + index] !== character} class:current={part.start + index === typed_text.length}>{'\u00a0'}</span>{/each}
        {:else}
          <span class="text_word">{#each [...part.value] as character, index}<span class:typed={part.start + index < typed_text.length} class:correct={part.start + index < typed_text.length && typed_text[part.start + index] === character} class:incorrect={part.start + index < typed_text.length && typed_text[part.start + index] !== character} class:current={part.start + index === typed_text.length}>{character}</span>{/each}</span>
        {/if}
      {/each}
    </div>
    <textarea value={typed_text} readonly={locked} oninput={handle_input} aria-label="Type the passage" spellcheck="false" autocomplete="off" autocapitalize="off"></textarea>
    {#if !typed_text}<div class="start_prompt">Type the passage <span>ENTER</span></div>{/if}
  </div>
  <div class="panel_footer"><span>{finished ? 'finished' : 'not finished yet'}</span><button class="btn btn-outline-light btn-sm" onclick={reset}>Restart</button></div>
</section>