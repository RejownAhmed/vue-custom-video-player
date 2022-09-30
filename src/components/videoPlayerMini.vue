
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
   * Show Video Controls
  */
  const videoControls = ref(true)
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
  const currentTime = ref('0:00')
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
  const hideControls = ()=>{
    // videoControls.value = false
  }
  let timer, idleTime;
  function resetTimer() {
      /*Set the timer to 0 */
      idleTime = 0
      /* Clear the previous interval */
      clearInterval(timer)
      videoControls.value = true
      /* Set a new interval */
      timer = setInterval(startIdleTimer, 1000)
      return
  }

  // Define the events that
  // would reset the timer
  onMounted(()=>{
    videoContainer.value.onmousemove = resetTimer;
    videoContainer.value.onmouseenter = resetTimer;
    videoContainer.value.onmousedown = resetTimer;
    videoContainer.value.ontouchstart = resetTimer;
    videoContainer.value.ontouchmove = resetTimer;
    videoContainer.value.onclick = resetTimer;
    videoContainer.value.ondblclick = resetTimer;
    videoContainer.value.onkeypress = resetTimer;
  })

  function startIdleTimer() {
    idleTime++
    // if(idleTime >= 5) videoControls.value = false
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
      videoControls.value = false
      return
    } else{
      videoEl.value.pause()
      playing.value = false
      showControls()
      return
    }
  }
  const loadSubtitle = ()=>{
    captionText.value = videoEl.value.textTracks[0].activeCues[0].text
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
    const duration = videoEl.value.duration
    const percent = videoEl.value.currentTime / duration
    /**
    * Below codes shows how we can update the progress bar position
    * Via the exposed template ref "ProgressPosition"
    * All other ways show at least one bug
    * See {@link customRange}
    */
    progressSlider.value.progressPosition = percent

    if (duration > 0) {
      for (let i = 0; i < videoEl.value.buffered.length; i++) {
        if (
          videoEl.value.buffered.start(videoEl.value.buffered.length - 1 - i) < videoEl.value.currentTime
        ) {
            progressSlider.value.bufferPosition = (videoEl.value.buffered.end(videoEl.value.buffered.length - 1 - i)) / duration
          break;
        }
      }
    }
    
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
      videoControls.value = true
  }
  </script>
  
  <template>
    <div class="mini-video-container" 
      :class="toggleClass"
      :ref="(e)=> videoContainer = e"
      :onmouseleave="hideControls"
    >
      <div class="progress-section">
        <div class="caption-container" v-show="showCaption">
          <div :ref="e => captionContainer = e">{{captionText}}</div>
        </div>
        <custom-range
            class="higher" 
            v-model="videoProgress" 
            show-buffer
            range-height="4px"
            range-container-height="4px"
            :ref="(e)=> progressSlider = e"
        >
        </custom-range>
      </div>
      <div class="video-controls-container" v-show="videoControls">
            <div class="controls">
              <div class="top">
                  <div class="left"></div>
                  <div class="right">
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
                  </div>
              </div>
              <div class="middle">
                  <button class="play-pause-btn" @click="togglePlay">
                      <svg v-if="!playing" viewBox="0 0 24 24">
                          <path fill="currentColor" d="M8,5.14V19.14L19,12.14L8,5.14Z" />
                      </svg>
                      <svg v-if="playing" viewBox="0 0 24 24">
                          <path fill="currentColor" d="M14,19H18V5H14M6,19H10V5H6V19Z" />
                      </svg>
                  </button>
              </div>
              <div class="bottom">
                  <div class="duration-container">
                      <span>{{currentTime}}</span>
                      /
                      <span>{{duration}}</span>
                  </div>
                  <div>
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
  
      </div>
      <h1 class="loader" v-if="loading">Loading....</h1>
      <video 
        :src="props.url" 
        :ontimeupdate="setProgress" 
        :onloadeddata="videoLoaded"
        :onended="videoEnded"
        :ref="(e)=> videoEl = e">
        <track kind="captions" type="text/vtt" srclang="en" :src="props.caption" default :oncuechange="loadSubtitle">
          Your browser does not support HTML5 video.
      </video>
  
    </div>
  </template>
  
  <style scoped>
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
    .mini-video-container {
      font-family: Arial, Helvetica, sans-serif;
      position: relative;
      width: 90%;
      max-width: 1000px;
      display: flex;
      justify-content: center;
      margin-inline: auto;
      background-color: black;
      overflow: hidden;
    }
  
    .mini-video-container.theater,
    .mini-video-container.full-screen {
      max-width: initial;
      width: 100%;
    }
  
    .mini-video-container.theater {
      max-height: 90vh;
    }
  
    .mini-video-container.full-screen {
      max-height: 100vh;
    }
  
    .progress-section{
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
    }
    .higher{
        position: relative;
        z-index: 1080;
    }
    video {
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
    .video-controls-container{
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        z-index: 100;
        width: 100%;
        color: #fff;
        background-color: rgba(0,0,0,0.3);
        padding: 0px 10px;
        box-sizing: border-box;
        display: block;
    }
    .controls{
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      height: 100%;
    }
    .top{
        width: 100%;
        display: flex;
        justify-content: space-between;
        align-content: flex-start;
    }
    .top .left, .top .right{
      display: inline-flex;
    }
    .middle{
        display: inline-flex;
        justify-content: center;
        align-items: center;
    }
    .middle button{
        height: 40px !important;
        width: 40px !important;
    }
    .bottom{
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
    }
    .video-controls-container button {
        background: none;
        border: none;
        color: inherit;
        padding: 0;
        height: 24px;
        width: 24px;
        font-size: 1.1rem;
        cursor: pointer;
        opacity: .85;
        transition: opacity 150ms ease-in-out;
    }
    .video-controls-container button:hover {
        opacity: 1;
    }
    .controls {
        display: flex;
        gap: .5rem;
        padding: 0rem .2rem;
        align-items: center;
        margin-top: 0px;
        padding-top: 0px;
    }
    .duration-container {
        margin-top: -3px;
        color: #fff;
        display: flex;
        align-items: center;
        gap: .25rem;
        flex-grow: 1;
        margin-left: 5px;
        font-size: 12px;
    }
  
    .controls button.wide-btn {
      width: 35px;
      font-size: 14px;
    }
  
  
  </style>