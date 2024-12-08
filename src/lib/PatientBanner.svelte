<script lang="ts">
  import { onMount } from "svelte";

    export let accessToken: string 
    export let baseUrl: string
    let patientResource: Patient | undefined = undefined

    onMount(async ()=>{

    })

</script>

<pre>
    {accessToken}
</pre>