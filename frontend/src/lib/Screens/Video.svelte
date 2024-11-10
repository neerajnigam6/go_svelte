<script lang="ts">
    import Icon from "@iconify/svelte"
    import { onMount } from "svelte";

    export let params: any = {}
    let width = 900
    let vid = [{ "description" : "Big Buck Bunny tells the story of a giant rabbit with a heart bigger than himself. When one sunny day three rodents rudely harass him, something snaps... and the rabbit ain't no bunny anymore! In the typical cartoon tradition he prepares the nasty rodents a comical revenge.\n\nLicensed under the Creative Commons Attribution license\nhttp://www.bigbuckbunny.org",
              "sources" : [ "https://commondatastorage.googleapis.com/gtv-videos-bucket/CastVideos/dash/BigBuckBunnyVideo.mp4" ],
              "subtitle" : "By Blender Foundation",
              "thumb" : "images/BigBuckBunny.jpg",
              "title" : "Big Buck Bunny"
            },
            { "description" : "The first Blender Open Movie from 2006",
              "sources" : [ "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4" ],
              "subtitle" : "By Blender Foundation",
              "thumb" : "images/ElephantsDream.jpg",
              "title" : "Elephant Dream"
            },
            { "description" : "HBO GO now works with Chromecast -- the easiest way to enjoy online video on your TV. For when you want to settle into your Iron Throne to watch the latest episodes. For $35.\nLearn how to use Chromecast with HBO GO and more at google.com/chromecast.",
              "sources" : [ "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4" ],
              "subtitle" : "By Google",
              "thumb" : "images/ForBiggerBlazes.jpg",
              "title" : "For Bigger Blazes"
            },
            { "description" : "Introducing Chromecast. The easiest way to enjoy online video and music on your TV—for when Batman's escapes aren't quite big enough. For $35. Learn how to use Chromecast with Google Play Movies and more at google.com/chromecast.",
              "sources" : [ "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerEscapes.mp4" ],
              "subtitle" : "By Google",
              "thumb" : "images/ForBiggerEscapes.jpg",
              "title" : "For Bigger Escape"
            }]
    let isPlaying = false
    let showControls = false
    let showControlsCounter = 0
    let container: HTMLDivElement
    let video: HTMLVideoElement
    let currentTime = "00:00"
    let videoProgressPercent: number = 0
    let leftSeekIcon: HTMLDivElement
    let rightSeekIcon: HTMLDivElement
    let playPauseIcon: HTMLDivElement

    onMount(() => {
      document.addEventListener("fullscreenchange", (e) => {
        setFullscreenData(!!document.fullscreenElement);
      });
    })
    
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

        if(!hours && !minutes) return `${remainingSeconds}`
        if(!hours) return `${minutes}:${remainingSeconds}`
        return `${hours}:${minutes}:${remainingSeconds}`;
    }

    function updateProgress(){
      const value = (video.currentTime / video.duration) * 100;
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

    function toggle_playback(element: HTMLDivElement) {
      showControlsFunc()
      if(!video) return
      if(video.paused || video.ended) {
        video.play()
        isPlaying = true
      }else {
        video.pause()
        isPlaying = false
      }
      toggleAnimation(element, 1)
    }

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
</script>

<div class="flex flex-col justify-center items-center h-screen w-screen" >
  <div class="w-fit h-fit relative" bind:this={container} on:focus={showControlsFunc}
  on:mousemove={showControlsFunc}
  >
    <video on:timeupdate={updateProgress} class="rounded h-fit w-[{width}px]" bind:this={video}>
      <source src="{vid[1].sources[0]}" />
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
    <div class="right-2 top-2 h-5/6 w-16 flex flex-col items-center absolute" 
        id="status_pane" aria-label="side-pane">
      <div class="flex flex-col {showControls?'block':'hidden'}">
        <div class="rounded-full bg-yellow-500 bg-opacity-50 m-1">
          <i class="fa fa-pause text-lg text-white px-5 py-4 "></i>
        </div>
        <div class="mt-5">
          {#if (document?.fullscreenEnabled)}
            <div aria-label="fullscreen-button" on:click={fullscreen} class="flex flex-row justify-center hover:bg-gray-500 hover:bg-opacity-50 py-2 items-center text-white rounded-full ">
              <Icon icon="mdi:fullscreen" width="2em" />
            </div>
          {/if}
        </div>
      </div>
    </div>

    <!-- progress bar -->
    <div class="absolute bottom-0 w-full py-2 px-4">
      <div class="{showControls?'block':'hidden'} w-full h-full  flex flex-row justify-center items-center">
        <button on:click={(e) => seekPosition(e)} class="py-4 w-full" aria-label="seek-button">
          <div class="bg-gray-500 bg-opacity-50 w-full rounded h-2">
            <div class="bg-yellow-500 rounded h-2 w-0" style="width: {videoProgressPercent}%" ></div>
          </div>
        </button>
        <div class="text-white px-2">{ currentTime }</div>
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

</style>