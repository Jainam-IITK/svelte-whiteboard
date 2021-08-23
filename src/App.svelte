<script lang="ts">
  import Video from './Video.svelte';
  import Home from './Home.svelte';
  import { Router, Link, Route } from "svelte-navigator";  let name = "Jainam";

  import firebase from 'firebase/app';
  import 'firebase/firestore';

  const firebaseConfig = {
    apiKey: "AIzaSyCEu5d0rrOw7Mi_FSdR6DDw-XVuGXim_80",
    authDomain: "whiteboard-f4cac.firebaseapp.com",
    projectId: "whiteboard-f4cac",
    storageBucket: "whiteboard-f4cac.appspot.com",
    messagingSenderId: "531838782598",
    appId: "1:531838782598:web:323853a2ae3dddeae15507",
    measurementId: "G-L3152V973W"
  };

  
  if (!firebase.apps.length) {
    firebase.initializeApp(firebaseConfig);
  }
 

</script>
 

<div class="app">

  <Router>
    <Route exact path="/" primary = {false}><Home/></Route>
    <Route exact path="/:id" primary = {false}><Video/></Route>
  </Router>


</div>


<style>


  
</style>
