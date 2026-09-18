<script>
  import { format_time } from './typing_utils.js'

  export let wpm
  export let accuracy
  export let consistency
  export let elapsed
  export let started_at
  export let finished
</script>

<section class="stats" aria-label="Typing statistics">
  <div class="stat stat_primary col-6 col-lg-3"><span>words per minute</span><strong>{wpm}</strong></div>
  <div class="stat col-6 col-lg-3"><span>accuracy</span><strong>{accuracy}<b>%</b></strong></div>
  <div class="stat col-6 col-lg-3"><span>consistency</span><strong>{consistency}<b>%</b></strong></div>
  <div class="stat col-6 col-lg-3"><span>time</span><strong>{format_time(elapsed)}</strong><small>{finished ? 'FINISHED' : started_at ? 'ONGOING' : 'READY'}</small></div>
</section>
