<svelte:head>
   <script src="https://unpkg.com/peerjs@1.3.1/dist/peerjs.min.js"></script>
</svelte:head>

<script>
   
   let appState = "Start";
   let myId;
   let peerId;
   let width = 400;
   let peer;

   let connectionObject;

   function getStarted() {
      peer = new Peer();
      peer.on('open', function(id) {
         console.log('My peer ID is: ' + id);
         myId = id;
         appState = "Host Waiting";
         
      });

      peer.on('connection', function(conn) {
         console.log("Connection coming from ", conn);
         connectionObject = conn;
         appState = "Game";
         
      })
   }

   function connectToPeer() {
      peer = new Peer();
      peer.on('connection', function(conn) {
         console.log("connected");
         connectionObject = conn;
      })

      peer.on('open', function(id) {
         console.log("my id is: ", id);
         connectionObject = peer.connect(peerId);

         console.log("Connecting to ", peerId, connectionObject);
         appState = "Game";
         connectionObject.on('open', function() {
            
            // Receive messages
            connectionObject.on('data', function(data) {
               console.log('Received', data);
            });

            // Send messages
            connectionObject.send('Hello!');
         });

         connectionObject.on('error', function(err) {
            console.log(err);
         })

         connectionObject.on('data', function(data) {
            if (data == "click!") {
               handleOpponentClick();
            }
         })
      })
   }

   function handleGameClick() {
      console.log(width);
      connectionObject.send("click!");
      width += 10;
   }

   function handleOpponentClick() {
      width -= 10;
   }

   $: if (width >= 800) {
      alert("You won!");
   }

</script>

<main>
   <h1>Clicker game</h1>
   {#if appState == "Start"}
      <button on:click={getStarted}>Host a Game!</button>
      <p>Or connect to someone else's game</p>
      <input bind:value={peerId} placeholder="enter someone's id">
      <button on:click={connectToPeer}>Go!</button>
   {/if}
   {#if appState == "Host Waiting"}
      <h2>My id is: <bold>{myId}</bold></h2>
      <p>Share it to your friends!</p>
   {/if}
   {#if appState == "Game"}
      <div class="game">
         <div class="backgroundBar">

            <div style="width:{width}px" class="clickerBar"></div>
         </div> 
         <button class="gameButton" on:click={handleGameClick}>Click me!</button>
      </div>
   {/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

   .clickerBar {
      height: 100px;
      background-color: red;
   }

   .backgroundBar {
      margin-left:auto;
      margin-right:auto;
      width: 800px;
      height: 100px;
      background-color: rgb(133, 133, 187);
   }

   .gameButton {
      margin-top: 50px;
      height: 100px;
      width: 400px;
   }

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>