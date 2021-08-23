<script>
import { onMount } from "svelte";

	import { useParams } from "svelte-navigator";
    import firebase from "firebase/app";
    import 'firebase/firestore'; 
    //getting id of the current room from the url

    const firestore = firebase.firestore();

    const params = useParams() ;
    let webCamButton ;
    let callButton ; 
    let callInput;
    let answerButton;
    let webcamVideo;
    let remoteVideo;
    let hangupButton;
    const id = $params.id;
    let localStream = null;
    let remoteStream = null;

    //initialising the RTC Peerconnection 
    const servers = {
    iceServers: [
        {
        urls: ['stun:stun1.l.google.com:19302', 'stun:stun2.l.google.com:19302'],
        },
    ],
    iceCandidatePoolSize: 10,
    };
    const pc = new RTCPeerConnection(servers);
    console.log('compontnt initialised')

    onMount(()=>{              


        webCamButton.onclick = async() =>{
            localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
            remoteStream = new MediaStream();

            // Push tracks from local stream to peer connection
            localStream.getTracks().forEach((track) => {
                pc.addTrack(track, localStream);
            });

            // Pull tracks from remote stream, add to video stream
            pc.ontrack = (event) => {
                event.streams[0].getTracks().forEach((track) => {
                remoteStream.addTrack(track);
                });
            };

            webcamVideo.srcObject = localStream;
            remoteVideo.srcObject = remoteStream;

            callButton.disabled = false;
            answerButton.disabled = false;
            webCamButton.disabled = true;
        }

        callButton.onclick = async() =>{
            // Reference Firestore collections for signaling
            const callDoc = firestore.collection('calls').doc(`${id}`);
            const offerCandidates = callDoc.collection('offerCandidates');
            const answerCandidates = callDoc.collection('answerCandidates');
            
            callInput.value = callDoc.id;

            // Get candidates for caller, save to db
            pc.onicecandidate = (event) => {
                console.log('on iceCandidatePoolSize')
                event.candidate && offerCandidates.add(event.candidate.toJSON());
            };
            // Create offer
            const offerDescription = await pc.createOffer();
            await pc.setLocalDescription(offerDescription);
            console.log('offer created')
            const offer = {
                sdp: offerDescription.sdp,
                type: offerDescription.type,
             };

            await callDoc.set({ offer });
            console.log('offer set')
            // Listen for remote answer
            callDoc.onSnapshot((snapshot) => {
                const data = snapshot.data();
                if (!pc.currentRemoteDescription && data?.answer) {
                const answerDescription = new RTCSessionDescription(data.answer);
                pc.setRemoteDescription(answerDescription);
                console.log('document changed')
                }
            });

            // When answered, add candidate to peer connection
            answerCandidates.onSnapshot((snapshot) => {
                snapshot.docChanges().forEach((change) => {
                if (change.type === 'added') {
                    const candidate = new RTCIceCandidate(change.doc.data());
                    pc.addIceCandidate(candidate);
                    console.log('ansercandidates added')
                }
                });
            });

            hangupButton.disabled = false;

        }

        // 3. Answer the call with the unique ID
        answerButton.onclick = async () => {
        const callId = callInput.value;
        const callDoc = firestore.collection('calls').doc(callId);
        const answerCandidates = callDoc.collection('answerCandidates');
        const offerCandidates = callDoc.collection('offerCandidates');

        pc.onicecandidate = (event) => {
            event.candidate && answerCandidates.add(event.candidate.toJSON());
        };

        const callData = (await callDoc.get()).data();

        const offerDescription = callData.offer;
        await pc.setRemoteDescription(new RTCSessionDescription(offerDescription));

        const answerDescription = await pc.createAnswer();
        await pc.setLocalDescription(answerDescription);

        const answer = {
            type: answerDescription.type,
            sdp: answerDescription.sdp,
        };

        await callDoc.update({ answer });

        offerCandidates.onSnapshot((snapshot) => {
            snapshot.docChanges().forEach((change) => {
            console.log(change);
            if (change.type === 'added') {
                let data = change.doc.data();
                pc.addIceCandidate(new RTCIceCandidate(data));
            }
            });
        });
        };
    })


    
    

</script>

<body>
    <h2>1. Start your Webcam</h2>

    <div class="videos">
      <span>
        <h3>Local Stream</h3>
        <video id="webcamVideo" bind:this = {webcamVideo} autoplay playsinline></video>
      </span>
      <span>
        <h3>Remote Stream</h3>
        <video id="remoteVideo"bind:this = {remoteVideo} autoplay playsinline></video>
      </span>


    </div>

    <button id="webcamButton" bind:this ={webCamButton} onclick={async() => await handleWebcamClick()}>Start webcam</button>
    <h2>2. Create a new Call</h2>
    <button id="callButton" bind:this={callButton} disabled>Create Call (offer)</button>

    <h2>3. Join a Call</h2>
    <p>Answer the call from a different browser window or device</p>
    
    <input id="callInput" bind:this={callInput} />
    <button id="answerButton" bind:this={answerButton} disabled>Answer</button>

    <h2>4. Hangup</h2>

    <button id="hangupButton" bind:this={hangupButton} disabled>Hangup</button>


  </body>



<style>

</style>