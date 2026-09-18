<script>
  export let file_name
  export let on_load
</script>

<label class="upload_control btn btn-outline-light">
  <span><strong>Upload a file</strong><small>{file_name}</small></span>
  <input type="file" accept=".txt,text/plain" onchange={on_load} />
</label>