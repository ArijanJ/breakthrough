<script>
    let videoElement

    let currentBackgroundColor = 'gray' // neutral, otherwise red or green

    const challenges = [
        // "jin_1break_s30_w20.mp4",
        // "jin_1break_s30_w20.mp4",
        "king_1break_s30_w20.mp4",
        "king_2break_s30_w20.mp4",
        "king_1+2break_s30_w20.mp4",
        "dragunov_1+2break_s71_w20.mp4",
        "dragunov_1break_s70_w20.mp4",
        "dragunov_2break_s66_w20.mp4"
    ]

    const preferences = {
        characters: [
            { name: "king", enabled: true, },
            { name: "dragunov", enabled: true }
        ]
    }

    let currentChallenge = {
        startTime: undefined,
        throwStartFrame: undefined,
        throwBreak: undefined,
        breakWindow: undefined, // 20 frame window to break, e.g.
    }

    function playChallenge(filepath) {
        videoElement.src = filepath
        videoElement.type = 'video/mp4'

        videoElement.load()

        // 60fps video titled for example:
        //      "jin_"  + "1break_"  +  "s30_w20.mp4"
        let [ character, break_button, start, window ] = filepath.split('_')
        console.log(filepath.split('_'))
        currentChallenge.throwStartFrame = start.substr(1) // remove 's' from start frame
        currentChallenge.throwBreak = break_button.replace('break','')
        currentChallenge.breakWindow = window.split('.')[0].substr(1) // remove file extension and 'w' from window
        videoElement.play(); // new timing gets assigned in onPlay
    }

    function playRandomChallenge() {
        let validChallenges = challenges.filter(challenge => {
            for (let character of preferences.characters) {
                if (challenge.startsWith(character.name) && character.enabled) {
                    return true
                }
            }
            return false
        })

        playChallenge(validChallenges[Math.floor(Math.random() * validChallenges.length)])
    }

    function onPlay(e) {
        currentChallenge.startTime = performance.now()
        console.log("video started playing at", currentChallenge.startTime);
        console.log('challenge:', currentChallenge)
    }

    function onEnd(e) {
        playRandomChallenge()
    }

    function onBreakAttempt(e) {
        let now = performance.now()
        console.log("break attempt at", now);

        let button = (() => {
            for (let [button, keys] of Object.entries(binds)) {
                if (keys.includes(e.key)) {
                    return button
                }
            }
            return undefined
        })()

        if(!button) {
            console.log("invalid binding")
            return
        }
        console.log('pressed ', button)

        let timeTaken = now - currentChallenge.startTime
        let framesSinceStart = timeTaken/1000*60
        let framesTakenToBreak = framesSinceStart - currentChallenge.throwStartFrame
        console.log("frames taken to break", framesTakenToBreak);

        console.log(currentChallenge)
        if (button === currentChallenge.throwBreak) {
            if (framesTakenToBreak <= currentChallenge.breakWindow &&
                framesSinceStart >= currentChallenge.throwStartFrame)
                currentBackgroundColor = 'green'
            else // mistimed
                currentBackgroundColor = 'yellow'
        } else /* complete failure */ {
            currentBackgroundColor = 'red'
        }

        playRandomChallenge()
    }

    let binds = {
        "1": ["u"],
        "2": ["i"],
        "1+2": ["o"]
    }

    let handleKeypress = (e) => {
        if (["Enter", " "].includes(e.key)) {
            playRandomChallenge()
        }
        else {
            onBreakAttempt(e)
        }
    }
</script>

<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />

<div id="videoContainer" on:keydown={handleKeypress} style="background-color: {currentBackgroundColor}">
    <!-- svelte-ignore a11y-media-has-caption -->
    <video bind:this={videoElement} on:loadeddata={onPlay} on:ended={onEnd}>
        Your browser does not support the video tag.
    </video>
</div>

{#if currentChallenge.startTime === undefined}
<div style="position: absolute; top: 50%; left: 50%; transform: translateX(-50%); z-index: 1;">
    <div style="display: flex; flex-direction: column; justify-content: center">
        {#each preferences.characters as character}
            <label>
                <input type="checkbox" bind:checked={character.enabled} />
                {character.name.charAt(0).toUpperCase() + character.name.slice(1)}
            </label>
        {/each}
        <button on:click={playRandomChallenge}>Start training</button>
    </div>
</div>
{/if}

<style>
    /* Container for the video */
    #videoContainer {
        margin: 0;
        padding: 0;
        position: relative;
        width: 100%;
        height: 100%;
        overflow: hidden; /* Hide any overflow */
    }

    /* Style the video to occupy 100% of the container */
    video {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 100%;
        height: 100%;
        object-fit: cover; /* Scale the video to cover the container */
        transform: translate(-50%, -50%); /* Center the video */
    }
</style>
