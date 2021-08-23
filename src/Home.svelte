

<script lang="typescript">
    
    import {Link} from 'svelte-navigator';
    import firebase from 'firebase/app';
    import 'firebase/firestore';
    import { onMount } from 'svelte';
    
    let id = null;
    onMount(()=>{
        const firestore = firebase.firestore();
        const callDoc = firestore.collection('calls').doc();
        id =  callDoc.id;
    })

  </script>
  


<main>

    {#if id != null}
    <Link to={`/${id}`} > <p>go to page</p></Link>
    {:else}
    <p>Loading...</p>
    {/if}
  </main> 
  
  <style>
    
  </style>