<script lang="ts">
    import Icon from "@iconify/svelte"
    import { onMount } from "svelte";

    export let params: any = {}
    export let width = 900
    export let autoplay = false
    export let src: string | string[] | Quality | Quality[]
    let localSrc: string[];
    
    type Quality = {
      quality: string,
      src: string[]
    }

    let isPlaying = false

    // if multiple quality options are available for this video
    let localQuality = []
    let currentQuality;
    let showControls = true
    let ended = false
    let showControlsCounter = 0
    let container: HTMLDivElement
    let video: HTMLVideoElement
    let currentTime = "00:00:00"
    let totalTime = "00:00:00"
    let videoProgressPercent: number = 0
    let leftSeekIcon: HTMLDivElement
    let rightSeekIcon: HTMLDivElement
    let playPauseIcon: HTMLDivElement

    onMount(() => {
      setLocalSrc()
      console.log(localSrc)
      console.log(localQuality)
      document.addEventListener("fullscreenchange", (e) => {
        setFullscreenData(!!document.fullscreenElement);
      });


    })

    function setLocalSrc() {
      if(src == null) return;
      if(typeof(src) == 'object') {
        console.log("src is object")
        // src is an array or QualityObject
        if(Array.isArray(src)){
          // src is array of strings or array of quality object
          // @ts-ignore
          if(src.length == 0) return;
          if(typeof(src[0]) == 'string'){
            localSrc = src
          }else{
            localSrc = src[0].src
            currentQuality = src[0]
            localQuality = src
          }
        }else{
          localSrc = src.src
          currentQuality = src
          localQuality = [src]
        }
      }else{
        localSrc = [src]
      }
    }

    function toggleQuality() {

    }

    let _lastVolume = 0;
    let volume = 50;

    function onKeyDown(event: KeyboardEvent) {
      if(event.repeat) return
      switch(event.key) {
        case "f":
          fullscreen()
          break;

        case " ":
          toggle_playback(playPauseIcon)
          break; 

        case "ArrowRight":
          seek(false, 5, rightSeekIcon)
          break;
        
        case "ArrowLeft":
          seek(true, -5, leftSeekIcon)
          break

        case "m":
          if(_lastVolume > 0){
            setVolume(_lastVolume)
            _lastVolume = 0
          }else {
            _lastVolume = volume
            setVolume(0)
          }
          
          
      }
    }
    
    function toggleAnimation(elem: HTMLDivElement, animation_duration: number = 1) {
      elem.style.display = "block"
      elem.style.animation = `fadeInOut ${animation_duration}s ease-in-out`
      setTimeout(() => {
        elem.style.display = "none"
      }, animation_duration * 1000)
    }

    function formatSeconds(seconds: number) {
        seconds = Math.round(seconds)
        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);
        const remainingSeconds = seconds % 60;

        if(!hours && !minutes) return `00:00:${remainingSeconds}`
        if(!hours) return `00:${minutes}:${remainingSeconds}`
        return `${hours}:${minutes}:${remainingSeconds}`;
    }

    function updateProgress(){
      const value = (video.currentTime / video.duration) * 100;
      if(video.ended || value > 95) {
        ended = true
      }else {
        ended = false
      }
      currentTime = formatSeconds(video.currentTime)
      videoProgressPercent = value
    }

    function setFullscreenData(state: boolean){
      video.setAttribute('data-fullscreen', (!!state).toString())
      if(state){
        container.setAttribute("class", "w-screen h-screen relative")
        video.setAttribute("class", "w-screen h-screen relative")
      }else {
        container.setAttribute("class", "w-fit h-fit relative")
        video.setAttribute("class", `w-[${width}px] h-fit relative`)
      }
    }

    function fullscreen(){
      showControlsFunc()
      if (document.fullscreenElement !== null) {
        // The document is in fullscreen mode
        document.exitFullscreen();
        setFullscreenData(false);
      } else {
        // The document is not in fullscreen mode
        container.requestFullscreen();
        setFullscreenData(true);
      }
    }

    function seek(seekLeft: boolean, time: number, element: HTMLDivElement) {
      showControlsFunc()
      if(seekLeft) {
        // seek left
        if(video.currentTime > 5) {
          video.currentTime -= 5
        }
        toggleAnimation(element, 1);
      } else {
        // seek right
        video.currentTime += 5
        toggleAnimation(element, 1);
      }
    }

    function seekPosition(e: MouseEvent){
      showControlsFunc()
      if(!e.currentTarget) return
  
      let rect = e.currentTarget.getBoundingClientRect();
      const w = (rect.right - rect.left)
      const pos = e.clientX - rect.left
      const clickPercent = pos/w
      let toSeekAt = video.duration * clickPercent
      console.log(toSeekAt)
      video.currentTime = toSeekAt
      // console.log(w/(e.clientX) * 100)
    }

    function setVolume(vol: number) {
      console.log("volume"  + volume)
      console.log(vol)
      video.volume = vol/100
      volume = vol 
    }

    function toggle_playback(element: HTMLDivElement) {
      showControlsFunc()
      if(!video) return
      totalTime = formatSeconds(video.duration)
      if(video.paused || video.ended) {
        video.play()
        isPlaying = true
      }else {
        video.pause()
        isPlaying = false
      }
      toggleAnimation(element, 1)
    }

    // temporarily display side pane and progress bar
    function showControlsFunc() {
      showControls = true
      showControlsCounter += 1
      setTimeout(() => {
        showControlsCounter -= 1
        if(showControlsCounter == 0){
          showControls = false
        }
      }, 3000)
    }

    function restartVideo() {
      video.currentTime = 0
      video.play()
    }
</script>

<svelte:window on:keydown={onKeyDown} />

<div class="flex flex-col h-fit w-fit" >
  <div class="w-fit h-fit relative" bind:this={container} on:focus={showControlsFunc}
      on:mousemove={showControlsFunc}
      role="presentation"
  >
    <video {autoplay} on:timeupdate={updateProgress} class="rounded h-fit" width="{width}" bind:this={video}
      
    >
      {#each localSrc as s}
        <source src="{s}" />
      {/each}
    </video>
    
    <div class="left-0 bottom-0 absolute h-full w-full p-0 m-0 flex flex-row" id="controls">
      <button on:click={(e) => seek(true, 5, leftSeekIcon)} aria-label="left-seek-button "
         class="flex flex-col items-center justify-center text-white  h-full w-1/3 m-0 p-0">
      <div bind:this={leftSeekIcon} aria-label="left-seek-icon" class="rounded-full bg-yellow-500 px-5 py-4 bg-opacity-50 hidden">
        <i class="fa fa-angles-left text-4xl"></i>
      </div>
      </button>
      <button class="flex flex-col text-white justify-center items-center h-full w-1/3 m-0 p-0"
       on:click={(e) => toggle_playback(playPauseIcon)} aria-label="play-pause-button">
        <div bind:this={playPauseIcon} aria-label="play-pause-icon" class="rounded-full bg-yellow-500 { isPlaying ? 'px-4 pl-5' : 'px-5'} py-4 bg-opacity-50 hidden text-center">
          <i class="fa { isPlaying ? 'fa-play' : 'fa-pause'} text-3xl"></i>
        </div>
      </button>
      <button on:click={(e) => seek(false, 5, rightSeekIcon)} aria-label="left-seek-button "
        class="flex flex-col items-center justify-center text-white h-full w-1/3 m-0 p-0">
        <div bind:this={rightSeekIcon} aria-label="right-seek-icon" class="rounded-full bg-yellow-500 px-4 py-4 bg-opacity-50 hidden">
          <i class="fa fa-angles-right text-4xl"></i>
        </div>
     </button>
    </div>
    
    <!-- status pane -->
    <div class="right-0 top-1 h-5/6 w-16 flex flex-col items-center absolute" 
        id="status_pane" aria-label="side-pane">
      <div class="flex flex-col {showControls?'block':'hidden'}">
        <!-- restart/play/pause -->
        {#if ended}
          <button class="rounded-full bg-yellow-500 bg-opacity-50 my-1" on:click={restartVideo}>
            <i class="fa fa-refresh text-lg text-whitew-14 h-14 "></i>
          </button>
        {:else}
          <button class="rounded-full bg-yellow-500 bg-opacity-50 my-1" on:click={toggle_playback}>
            <i class="fa { isPlaying ? 'fa-play w-12 h-12' : 'fa-pause w-12 h-12'} text-lg text-white  py-4 "></i>
          </button>
        {/if}


        <!-- fullscreen -->
        <div class="">
          {#if (document?.fullscreenEnabled)}
            <div aria-label="fullscreen-button" on:click={fullscreen} class="flex flex-row justify-center hover:bg-gray-500 bg-opacity-50 bg-gray-700 my-1 w-12 h-12 items-center text-white rounded-full ">
              <Icon icon="mdi:fullscreen" width="2em" />
            </div>
          {/if}

          <!-- quality selector if available -->
          {#if localQuality.length > 0}
            <div aria-label="qulity-button" on:click={toggleQuality} class="flex flex-row justify-center hover:bg-gray-500 bg-opacity-50 bg-gray-700 my-1 w-12 h-12 items-center text-white rounded-full ">
              <p class="text-sm">{currentQuality.quality}</p>
            </div>
          {/if}
          <!-- volume -->
          <div class="flex flex-row justify-center h-16">
            <input type="range" orient="vertical" bind:value={volume}  on:change={(event) => setVolume(volume)}/></div>
        </div>
      </div>
    </div>

    <!-- progress bar -->
    <div class="absolute bottom-0 w-full py-2 px-4">
      <div class="{showControls?'block':'hidden'} w-full h-full  flex flex-row justify-center items-center">
        <button on:click={(e) => seekPosition(e)} class="py-4 w-full" aria-label="seek-button">
          <div class="bg-gray-500 bg-opacity-20 w-full h-1 group hover:h-2 transition ease-in-out duration-200">
            <div class="bg-yellow-500 h-1 w-0 group-hover:h-2 transition ease-in-out duration-200" style="width: {videoProgressPercent}%" ></div>
          </div>
        </button>
        <div class="text-white px-2">{ currentTime }/{totalTime}</div>
      </div>
    </div>
  </div>
  
  
</div>

<style>
  @keyframes fadeInOut {
    0% { opacity: 0; }
    30% { opacity: 0.1;}
    50% { opacity: 0.3; } 
    70% { opacity: 0.5; }
    90% { opacity: 1; }
  }

  input[type=range][orient=vertical] {
      writing-mode: vertical-lr;
      direction: rtl;
      appearance: slider-vertical;
      width: 8px;
      vertical-align: bottom;
  }

  button {
    outline: 2px solid transparent;
    outline-offset: 2px;
  }
</style>