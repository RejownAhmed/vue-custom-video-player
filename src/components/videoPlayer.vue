
<script setup>
  import {ref, onBeforeMount, watch, onMounted, computed} from 'vue'
  import customRange from './customRangeController.vue'

  const props = defineProps(
    {
      url: String,
      caption: String
    }
  )
  /**
   * Component Selector - Progress Slider
  */
  const videoContainer = ref()
  /**
   * Selector - Video Element When mounted
  */
  const videoEl = ref()
  /**
   * Video Caption Container
  */
  const captionContainer = ref()
  /**
   * Component Selector - Progress Slider
  */
  const progressSlider = ref()
  /**
   * Component Selector - Volume Slider
  */
  const volumeSlider = ref()
  /**
   * Show Video Controls
  */
  const showControls = ref(true)
  /**
   * Initial Volume
  */
  const volume = ref(0.5)
  /**
   * Initial Progress
  */
  const videoProgress = ref(0)
  /**
   * Video Duration
  */
  const duration = ref('0:00:00')
  /**
   * currentTime ontimeupdate of the file
  */
  const currentTime = ref('0:00:00')
  /**
   * Video Caption
  */
  const captionText = ref('')
  /**
   * Loading or Not 
  */
  const loading = ref(true)
  /**
    * Playing or Not
  */
  const playing =  ref(false)
  /**
    * Playback Speed
  */
  const playbackspeed =  ref(1)
  /**
    * Muted or Not
  */
  const muted = ref(false)
  /**
   * Volume Level
  */
    const volumeLevel = ref()
  /**
    * Theater Mode
  */
  const theater = ref(false)
  /**
    * Fullscreen Mode
  */
  const fullscreen = ref(false)
  /**
    * Picture in Picture Mode
  */
  const miniscreen = ref(false)
  /**
    * Video Caption
  */
  const showCaption = ref(false)
  /**
   * Show/Hide Video Controls
  */
  function mouseEntered() {
    showControls.value = true
  }
  function mouseLeft() {
    playing.value ? showControls.value = false : showControls.value = true
  }
  /**
   * Video Loaded in the DOM
  */
  const videoLoaded = ()=>{
    const videoWidth = videoEl.value.getBoundingClientRect().width
    let captionFontSize = 12
    videoEl.value.textTracks[0].mode = "hidden"//Hide Subtitle
    if(videoWidth > 400) captionFontSize = captionFontSize + Math.ceil((videoWidth - 400) / 100)
    captionContainer.value.setAttribute('style', `font-size: ${captionFontSize+"px"}`)

    setTimeout(()=>{
      // videoEl.value.play()
      // playing.value = true
      loading.value = false //Hide loader
    }, 200)
    setDuration() //Set duration
  }
  //PlayPause
  const togglePlay = ()=> {
    if(videoEl.value.paused) {
      videoEl.value.play()
      playing.value = true
      showControls.value = false
      return
    } else{
      videoEl.value.pause()
      playing.value = false
      showControls.value = true
      return
    }
  }
  const loadSubtitle = ()=>{
    captionText.value = videoEl.value.textTracks[0].activeCues[0].text
    

  }
  /**
   * Check if the volume is set already from localstorage
   * and set the new volume 
  */
  onBeforeMount(()=>{
    if( volume.value >= 0.5 ) {
          volumeLevel.value = 'high'
    } else {
        volumeLevel.value = 'low'
    }
  })
  onMounted(()=>{
    videoEl.value.volume = volume.value //Set initial volume
    if (volume.value == 0) {
      videoEl.value.muted == true
      muted.value = true
    } //If muted mute the video
  })
  // Gets a varibale and watches for changes 
    watch(volume, (newVolume, prevVolume)=>{//set volume
    videoEl.value.volume = newVolume 
    if( volume.value > 0 ) {//if user changes volume after muting unmute the video and set new volume
        muted.value = false
        videoEl.value.muted = false
    }
  })
  /**
   * Checks if user has changed device volume and sets the volume
   */
  const updateVolume = () =>{//On Volume Change
      volume.value = videoEl.value.volume
      volumeSlider.value.progressPosition = videoEl.value.volume
      if( videoEl.value.muted || volume.value == 0 ) {
        muted.value = true
      }else if( volume.value >= 0.5 ) {
        volumeLevel.value = 'high'
        muted.value = false
      } else{
        volumeLevel.value = 'low'
        muted.value = false
      }

  }
  /**
    * Mute/Unmute method
    * Also sets the volume to a lower level( > 0) if video is unmuted 
    * while the volume was zero
    * So that user can at least understand that the video is unmuted
  */
  const toggleMute = ()=> {
    console.log(videoEl.value.muted);
    videoEl.value.muted = !videoEl.value.muted
    if (videoEl.value.muted) {
      if (volume.value == 0) {
        videoEl.value.volume = 0.1
        volume.value = 0.1
        volumeSlider.value.progressPosition = 0.1//Update the porgress bar position
      } else {
        videoEl.value.volume = volume.value
      }

    }

  }
  /**
   * Gets the duration of the video/audio file
   * sets the duration to variable
  **/
  const setDuration = ()=>{
    duration.value = formatDuration(videoEl.value.duration)//Get formatted time and set duration
  }
  const leadingZeroFormatter = new Intl.NumberFormat(undefined, {
    minimumIntegerDigits: 2,
  })
  function formatDuration(time) {//Format Duration 
    const seconds = Math.floor(time % 60)
    const minutes = Math.floor(time / 60) % 60
    const hours = Math.floor(time / 3600)
    if (hours === 0) {
      return `${minutes}:${leadingZeroFormatter.format(seconds)}`
    } else {
      return `${hours}:${leadingZeroFormatter.format(minutes)}:${leadingZeroFormatter.format(seconds)}`
    }
  }
  /**
   * Watches for videoProgress changes 
   * when user drag/clicks the slider
  */
  watch(videoProgress, (progress, prevProgress)=>{//set video progress
    videoEl.value.currentTime = progress * videoEl.value.duration 
  })
  const setProgress = ()=>{
    currentTime.value = formatDuration(videoEl.value.currentTime)
    const percent = videoEl.value.currentTime / videoEl.value.duration
    /**
    * Below codes shows how we can update the progress bar position
    * Via the exposed template ref "ProgressPosition"
    * All other ways show at least one bug
    * See {@link customRange}
    */
    progressSlider.value.progressPosition = percent
    
  }



  /**
   * Skip to a certain position
  */
  const skip = (type, skipPosition)=> {
    if (type == 'video') {//Skip Video Progress
        let newPosition = videoEl.value.currentTime + skipPosition
        if (newPosition >= videoEl.value.duration) {
            videoEl.value.currentTime = videoEl.value.duration - 1
        } else if(newPosition <= 0){
            videoEl.value.currentTime = 0
            volume.value = 0.1
        } else {
            videoEl.value.currentTime = newPosition
        }
      
    } 
    if (type == 'volume') {//Skip Volume
        let newVolume = videoEl.value.volume + skipPosition
        if (newVolume >= 1) {
            videoEl.value.volume = 1
        }else if (newVolume <= 0) {
            videoEl.value.volume = 0
        } else{
            videoEl.value.volume = newVolume
        }
    } 

  }

  /**
   * Toggle Screen Mode
  */
  const toggleTheaterMode = ()=> {//Theater Mode
    videoContainer.value.classList.toggle("theater")
    theater.value = !theater.value
  }

  const toggleFullScreenMode = ()=> {//Full Screen Mode
    if (!document.fullscreenElement) {
        if (videoContainer.value.requestFullscreen) {
            videoContainer.value.requestFullscreen()
        } else if (videoContainer.value.webkitRequestFullscreen) {/* Safari */
            videoContainer.value.webkitRequestFullscreen()
        } else if (videoContainer.value.msRequestFullscreen) {/* IE11 */
            videoContainer.value.msRequestFullscreen()
        } else if (videoContainer.value.mozRequestFullscreen) {/* Mozilla Firefox */
            videoContainer.value.mozRequestFullscreen()
        }
        fullscreen.value = true

    } else if(document.exitFullscreen){
        if(document.exitFullscreen) {
          document.exitFullscreen();
        } else if(document.mozCancelFullScreen) {/* Mozilla Firefox */
          document.mozCancelFullScreen()
        } else if(document.webkitExitFullscreen) {/* Safari */
          document.webkitExitFullscreen()
        } else if(document.msExitFullscreen) {/* IE11 */
          document.msExitFullscreen()
        }
        fullscreen.value = false

    }
  }

  if (document.addEventListener)
  {
    document.addEventListener('fullscreenchange', exitHandler)
    document.addEventListener('mozfullscreenchange', exitHandler)
    document.addEventListener('MSFullscreenChange', exitHandler)
    document.addEventListener('webkitfullscreenchange', exitHandler)
  }
  function exitHandler(e) {//Exit full screen
    if (!document.webkitIsFullScreen && !document.mozFullScreen && !document.msFullscreenElement){
      fullscreen.value = false
    }
  }

  const toggleClass = computed(()=>{//Set classed for theater/full-screen mode
    if (fullscreen.value) {
      return 'full-screen'
    } 
    if (theater.value) {
      return 'theater'
    }  
    return ''
  })
  //Playback Speed
  const changePlaybackSpeed = ()=> {
      let newPlaybackRate = videoEl.value.playbackRate + 0.25
      if (newPlaybackRate > 2) newPlaybackRate = 0.25
      videoEl.value.playbackRate = newPlaybackRate
      playbackspeed.value = newPlaybackRate
  }

  /**
   * Fire when video finished playing
  */
  function videoEnded(){
      playing.value = false//Change playing icon to pause
  }
  //Keyboard Shortcuts
  document.addEventListener("keydown", e => {
      const tagName = document.activeElement.tagName.toLowerCase()
      if (tagName === "input") return 
      switch (e.key.toLowerCase()) {
        case " ":
          if (tagName === "button") return
        case "k":
          togglePlay()
          break
        case "f":
          toggleFullScreenMode()
          break
        case "t":
          toggleTheaterMode()
          break
        case "i":
          toggleMiniPlayerMode()
          break
        case "m":
          toggleMute()
          break
        case "arrowleft":
        case "j":
          skip('video', -5)
          break
        case "arrowright":
        case "l":
          skip('video', 5)
          break
        case "arrowup":
          skip('volume', 0.1)
          break
        case "arrowdown":
          skip('volume', -0.1)
          break
        case "c":
          toggleCaptions()
          break
      }
    })

</script>

<template>
  <div class="video-container" 
    :class="toggleClass"
    :ref="(e)=> videoContainer = e"
    :onmouseover="mouseEntered"
    :onmouseout="mouseLeft">
    <div class="video-bottom-section">
      <div class="caption-container" v-show="showCaption">
        <div :ref="e => captionContainer = e">{{captionText}}</div>
      </div>
      <div class="video-controls-container" v-show="showControls">
          <custom-range 
            v-model="videoProgress" 
            expand-on-hover
            :ref="(e)=> progressSlider = e"
          >
            <img class="preview-img">
          </custom-range>
        <div class="controls">
          <button class="play-pause-btn" @click="togglePlay">
            <svg v-if="!playing" viewBox="0 0 24 24">
              <path fill="currentColor" d="M8,5.14V19.14L19,12.14L8,5.14Z" />
            </svg>
            <svg v-if="playing" viewBox="0 0 24 24">
              <path fill="currentColor" d="M14,19H18V5H14M6,19H10V5H6V19Z" />
            </svg>
          </button>
          <div class="volume-container">
            <button @click="toggleMute">
              <svg v-if="!muted && volumeLevel == 'high'" viewBox="0 0 24 24">
                <path fill="currentColor" d="M14,3.23V5.29C16.89,6.15 19,8.83 19,12C19,15.17 16.89,17.84 14,18.7V20.77C18,19.86 21,16.28 21,12C21,7.72 18,4.14 14,3.23M16.5,12C16.5,10.23 15.5,8.71 14,7.97V16C15.5,15.29 16.5,13.76 16.5,12M3,9V15H7L12,20V4L7,9H3Z" />
              </svg>
              <svg v-if="!muted && volumeLevel == 'low'" viewBox="0 0 24 24">
                <path fill="currentColor" d="M5,9V15H9L14,20V4L9,9M18.5,12C18.5,10.23 17.5,8.71 16,7.97V16C17.5,15.29 18.5,13.76 18.5,12Z" />
              </svg>
              <svg v-if="muted" viewBox="0 0 24 24">
                <path fill="currentColor" d="M12,4L9.91,6.09L12,8.18M4.27,3L3,4.27L7.73,9H3V15H7L12,20V13.27L16.25,17.53C15.58,18.04 14.83,18.46 14,18.7V20.77C15.38,20.45 16.63,19.82 17.68,18.96L19.73,21L21,19.73L12,10.73M19,12C19,12.94 18.8,13.82 18.46,14.64L19.97,16.15C20.62,14.91 21,13.5 21,12C21,7.72 18,4.14 14,3.23V5.29C16.89,6.15 19,8.83 19,12M16.5,12C16.5,10.23 15.5,8.71 14,7.97V10.18L16.45,12.63C16.5,12.43 16.5,12.21 16.5,12Z" />
              </svg>
            </button>
            <custom-range 
              :ref="(e)=> volumeSlider = e"
              v-model="volume"
              class="volume-slider"
              progress-color="white"
              thumb-color="white"
              thumb-height="400%"
              always-show-thumb
              no-preview-bar>
            </custom-range>
          </div>
          <div class="duration-container">
            <div>{{currentTime}}</div>
            /
            <div>{{duration}}</div>
          </div>
          <button 
            @click="showCaption = !showCaption" 
            class="captions-btn"
            :style="showCaption ? {borderBottom:' 3px solid red'} : ''">
            <svg viewBox="0 0 24 24">
              <path fill="currentColor" d="M18,11H16.5V10.5H14.5V13.5H16.5V13H18V14A1,1 0 0,1 17,15H14A1,1 0 0,1 13,14V10A1,1 0 0,1 14,9H17A1,1 0 0,1 18,10M11,11H9.5V10.5H7.5V13.5H9.5V13H11V14A1,1 0 0,1 10,15H7A1,1 0 0,1 6,14V10A1,1 0 0,1 7,9H10A1,1 0 0,1 11,10M19,4H5C3.89,4 3,4.89 3,6V18A2,2 0 0,0 5,20H19A2,2 0 0,0 21,18V6C21,4.89 20.1,4 19,4Z" />
            </svg>   
          </button>
          <button class="speed-btn wide-btn" @click="changePlaybackSpeed">
            {{playbackspeed}}x
          </button>
          <button class="theater-btn" @click="toggleTheaterMode">
            <svg v-if="!theater" viewBox="0 0 24 24">
              <path fill="currentColor" d="M19 6H5c-1.1 0-2 .9-2 2v8c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2zm0 10H5V8h14v8z"/>
            </svg>
            <svg v-if="theater" viewBox="0 0 24 24">
              <path fill="currentColor" d="M19 7H5c-1.1 0-2 .9-2 2v6c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V9c0-1.1-.9-2-2-2zm0 8H5V9h14v6z"/>
            </svg>
          </button>
          <button class="fullscreen-btn" @click="toggleFullScreenMode">
            <svg v-if="!fullscreen" viewBox="0 0 24 24">
              <path fill="currentColor" d="M7 14H5v5h5v-2H7v-3zm-2-4h2V7h3V5H5v5zm12 7h-3v2h5v-5h-2v3zM14 5v2h3v3h2V5h-5z"/>
            </svg>
            <svg v-if="fullscreen" viewBox="0 0 24 24">
              <path fill="currentColor" d="M5 16h3v3h2v-5H5v2zm3-8H5v2h5V5H8v3zm6 11h2v-3h3v-2h-5v5zm2-11V5h-2v5h5V8h-3z"/>
            </svg>
          </button>
        </div>
      </div>

    </div>
    <h1 class="loader" v-if="loading">Loading....</h1>
    <video 
      :src="props.url" 
      :ontimeupdate="setProgress" 
      :onvolumechange="updateVolume" 
      :onloadeddata="videoLoaded"
      :onended="videoEnded"
      :ref="(e)=> videoEl = e"
      @click="togglePlay">
      <track kind="captions" type="text/vtt" srclang="en" :src="props.caption" default :oncuechange="loadSubtitle">
    	Your browser does not support HTML5 video.
    </video>

  </div>
</template>

<style>
  /** 
    * Hide all controls
  */
  ::-webkit-media-controls {
    display:none !important;
  }
  video::-webkit-media-controls {
    display:none !important;
  }
  video::-webkit-media-controls-enclosure {
    display:none !important;
  }
  /*
  *Video Loader
  */
  .loader{
    position: absolute;
    color: #fff;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  .video-container {
    font-family: Arial, Helvetica, sans-serif;
    position: relative;
    width: 90%;
    max-width: 1000px;
    display: flex;
    justify-content: center;
    margin-inline: auto;
    background-color: black;
  }

  .video-container.theater,
  .video-container.full-screen {
    max-width: initial;
    width: 100%;
  }

  .video-container.theater {
    max-height: 90vh;
  }

  .video-container.full-screen {
    max-height: 100vh;
  }

  video {
    width: 100%;
  }
  .video-bottom-section{
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 100;
    width: 100%;
  }
  .caption-container{
    width: 100%;
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .caption-container div{
    background: rgba(0,0,0,0.3);
    color: #fff;
    padding: 5px;
    max-width: 85%;
    display: inline-block;
    text-align: center;
  }
  .video-controls-container {
    color: white;
  }

  .video-controls-container::before {
    content: "";
    position: absolute;
    bottom: 0;
    background: linear-gradient(to top, rgba(0, 0, 0, .75), transparent);
    width: 100%;
    aspect-ratio: 6 / 1;
    z-index: -1;
    pointer-events: none;
  }
  .video-controls-container .controls {
    display: flex;
    gap: .5rem;
    padding: .25rem;
    align-items: center;
  }

  .video-controls-container .controls button {
    background: none;
    border: none;
    color: inherit;
    padding: 0;
    height: 30px;
    width: 30px;
    font-size: 1.1rem;
    cursor: pointer;
    opacity: .85;
    transition: opacity 150ms ease-in-out;
  }

  .video-controls-container .controls button:hover {
    opacity: 1;
  }

  .volume-container {
    display: flex;
    align-items: center;
  }

  .volume-slider {
    width: 0;
    transform-origin: left;
    transform: scaleX(0);
    transition: width 150ms ease-in-out, transform 150ms ease-in-out;
  }

  .volume-container:hover .volume-slider,
  .volume-slider:focus-within {
    width: 52px;
    transform: scaleX(1);
  }

  .duration-container {
    display: flex;
    align-items: center;
    gap: .25rem;
    flex-grow: 1;
  }

  .video-controls-container .controls button.wide-btn {
    width: 50px;
  }


</style>