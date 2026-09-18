<script>
  export let target_text
  export let typed_text
  export let locked
  export let finished
  export let reset
  export let handle_input

  let text_parts = [{ value: '', start: 0, is_space: false }]

  $: {
    let start = 0
    text_parts = target_text.split(/(\s+)/).map((value) => {
      const part = { value, start, is_space: /^\s+$/.test(value) }
      start += value.length
      return part
    })
  }
</script>

<section class="practice_panel">
  <div class="panel_heading"><span>Passage / {target_text.length} chars</span></div>
  <div class="typing_wrap" class:complete={finished}>
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