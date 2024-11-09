<script lang="ts">
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
    let video: HTMLVideoElement
    function toggle_playback() {
      if(!video) return
      if(video.paused || video.ended) {
        video.play()
      }else {
        video.pause()
      }
    }
</script>

<div class="flex flex-col justify-center items-center h-screen w-screen">
  <div class="w-fit h-fit relative">
    <video width="{width}" class="rounded" bind:this={video}>
      <source src="{vid[1].sources[0]}" />
    
    </video>  
    <div class="left-0 bottom-0 absolute h-full w-full p-0 m-0 flex flex-row" id="controls">
      <button class="flex flex-col text-white bg-green-200 bg-opacity-20 h-full w-1/3 m-0 p-0">left-seek</button>
      <button class="flex flex-col text-white bg-red-200 bg-opacity-20 h-full w-1/3 m-0 p-0" on:click={toggle_playback}>Play</button>
      <button class="flex flex-col text-white bg-green-200 bg-opacity-20 h-full w-1/3 m-0 p-0">right-seek</button>
    </div>
  </div>
  
</div>
