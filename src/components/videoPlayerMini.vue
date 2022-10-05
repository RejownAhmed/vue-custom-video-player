
<script setup>
  import {ref, onBeforeUnmount, watch, onMounted, computed} from 'vue'
  import customRange from './customRangeController.vue'

  const props = defineProps(
    {
      url: String,
      caption: String
    }
  )
  const showForward = ref(false)
  const showBackward = ref(false)
  const skippedForward = ref(0)
  const skippedBackward = ref(0)
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
    videoControls.value = false
  }

  let tapped = false
  const loadTouch = (e)=>{
    if(!tapped){ //if tap is not set, set up single tap
      tapped=setTimeout(function(){
          tapped=null
          //when single tapped
          resetTimer(e)
      },300);   //wait 300ms then run single click code
    } else {    //tapped within 300ms of last tap. double tap
      clearTimeout(tapped); //stop single tap callback
      tapped=null
      //when double tapped
      if (e.target.classList.contains('fastForward')) {
          fastForward()
      } else if (e.target.classList.contains('fastBackward')) {
          fastBackward()
      }
    }
    e.preventDefault()
  }

  let timer, idleTime;
  function resetTimer(e) {
      if(e.target.classList.contains('controls')) {
        e.stopPropagation()
        videoControls.value = false
      } else {
        videoControls.value = true
      }
      // console.log(e);
      /*Set the timer to 0 */
      idleTime = 0
      /* Clear the previous interval */
      clearInterval(timer)
      timer = setInterval(startIdleTimer, 1000)
      /* Set a new interval */
      return
  }
  // Define the events that
  // would reset the timer

  onBeforeUnmount(()=>{
    clearInterval(timer)// Clear the interval when unMounted
  })
  function startIdleTimer() {
    idleTime++
    if(idleTime >= 5) videoControls.value = false
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
  const videoWaiting = ()=>{
      loading.value = true
  }
  const videoPlaying = ()=>{
      loading.value = false
  }
  //PlayPause
  let toggleVideoControls;
  const togglePlay = ()=> {
    if(videoEl.value.paused) {
      videoEl.value.play()
      playing.value = true
      toggleVideoControls = setTimeout(()=> videoControls.value = false, 1000)
      return
    } else{
      videoEl.value.pause()
      clearTimeout(toggleVideoControls)
      loading.value = false //Hide loader(If enabled by onwaiting event)
      playing.value = false
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
    currentTime.value = formatDuration(videoEl.value.currentTime)//Get formatted current time
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
  let skippedForwardTimer, skippedBackwardTimer;
  const fastForward = ()=>{
    showForward.value = true
    skip(10)
    var hideForward = setTimeout(()=> showForward.value = false, 500)
    clearTimeout(skippedForwardTimer) //if clicked more than once in 1s
    //clear the timeout and increase the skippedForward value
    skippedForward.value += 10
    //if clicked only once make the skippedForward value to 0 after 1s
    skippedForwardTimer = setTimeout(()=> skippedForward.value = 0, 1000)
  }
  const fastBackward = ()=>{
    showBackward.value = true
    skip(-10)
    var hideBackward = setTimeout(()=> showBackward.value = false, 500)
    clearTimeout(skippedBackwardTimer) //if clicked more than once in 1s
    //clear the timeout and increase the skippedBackward value
    skippedBackward.value += 10
    //if clicked only once make the skippedBackward value to 0 after 1s
    skippedBackwardTimer = setTimeout(()=> skippedBackward.value = 0, 1000)
  }

  /**
   * Skip to a certain position
  */
 let skipped;
  const skip = (skipPosition)=> {//Skip Video Progress
        let newPosition = videoEl.value.currentTime + skipPosition
        if (newPosition >= videoEl.value.duration) {
            videoEl.value.currentTime = videoEl.value.duration - 1
        } else if(newPosition <= 0){
            videoEl.value.currentTime = 0
        } else {
            videoEl.value.currentTime = newPosition
        }
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
      <div class="skip-btns" :ontouchstart="loadTouch">
        <div class="left">
            <div class="fastBackward"></div>
            <button>
              <svg fill="#ccc" viewBox="0 0 24 24">
                <g>
                  <Transition name="forwardThree">
                    <path v-show="showBackward" d="M8.86,18.54V5.43L.54,12.07Zm-.73-1.49-6.42-5L8.13,7Z"/>
                  </Transition>
                  <Transition name="forwardTwo">
                    <path v-show="showBackward" d="M8.49,12.63l.37.28,7.28,5.66V5.46L8.86,11.27l-.73.58v.49ZM15.4,7v10.1L9,12.1Z"/>
                  </Transition>
                  <Transition name="forwardOne">
                    <path v-show="showBackward" d="M15.77,12.59l.37.29,7.32,5.69V5.46l-7.32,5.85-.37.29-.37.3v.41Zm7-5.61v10.1l-6.41-5Z"/>
                  </Transition>
                </g>
              </svg>
              <Transition name="skipText">
                <div v-show="showBackward">{{skippedBackward}}s</div>
              </Transition>
            </button>
        </div>
        <div class="right">
            <div class="fastForward"></div>
            <button>
                <svg fill="#ccc" viewBox="0 0 24 24" stroke="3px">
                  <g>
                      <Transition name="forwardThree">
                        <path v-show="showForward" d="M15.14,5.46V18.57l8.32-6.64ZM15.87,7l6.42,5-6.42,5.12Z"/>
                      </Transition>
                      <Transition name="forwardTwo">
                        <path v-show="showForward" d="M15.51,11.37l-.37-.28L7.86,5.43V18.54l7.28-5.81.73-.58v-.49ZM8.6,17V6.92l6.41,5Z"/>
                      </Transition>
                      <Transition name="forwardOne">
                        <path v-show="showForward" d="M8.23,11.41l-.37-.29L.54,5.43V18.54l7.32-5.85.37-.29.37-.3v-.41ZM1.28,17V6.92l6.41,5Z"/>
                      </Transition>
                    </g>
                </svg>
                <Transition name="skipText">
                  <div v-show="showForward">{{skippedForward}}s</div>
                </Transition>
            </button>
        </div>
      </div>
      <div class="progress-section">
          <div class="caption-container" v-show="showCaption">
            <div class="caption-text" :ref="e => captionContainer = e">{{captionText}}</div>
          </div>
          <custom-range
              class="higher" 
              v-model="videoProgress" 
              show-buffer
              always-show-thumb
              range-height="4px"
              range-container-height="4px"
              thumb-height="250%"
              :ref="(e)=> progressSlider = e">
          </custom-range>
      </div>
      <div class="video-controls-container" 
        v-show="videoControls"
        :ontouchstart="resetTimer" :ontouchmove="resetTimer"
      >
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
                          <p>{{currentTime}} / {{duration}}</p>
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
        :onwaiting="videoWaiting"
        :onplaying="videoPlaying"
        :onended="videoEnded"
        :ref="(e)=> videoEl = e">
        <track kind="captions" type="text/vtt" srclang="en" :src="props.caption" default :oncuechange="loadSubtitle">
          Your browser does not support HTML5 video.
      </video>
  
    </div>
  </template>
  
  <style lang="scss" scoped>
    .mini-video-container {
      -webkit-user-select: none;
      user-select: none;
      font-family: Arial, Helvetica, sans-serif;
      position: relative;
      width: 100%;
      max-width: 500px;
      max-height: 300px;
      display: flex;
      justify-content: center;
      margin-inline: auto;
      background-color: black;
      &.theater, &.full-screen {
        max-width: initial;
        width: 100%;
      }
      &.full-screen {
        max-height: 100vh;
      }
      &.theater {
        max-height: 90vh;
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
      .skip-btns{
        position: absolute;
        height: 100%;
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        .left, .right {
          position: relative;
          width: 40%;
          height: 100%;
          display: flex;
          justify-content: center;
          align-items: center;
          z-index: 10;
          button{
            background: transparent;
            border: none;
            outline: none;
            z-index: 3;
            display: inline-block;
            gap: 0.5rem;
            div{
              font-size: 16px;
              color: #ccc;
              overflow: hidden;
              width: 35px;
              height: 20px;
            }
            svg{
              height: 24px;
              width: auto;
            }

          }
        }
        .left {
          .fastBackward{
              position: absolute;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
              background: transparent;
              z-index: 10;
          }
        }
        .right{
          .fastForward{
              position: absolute;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
              background: transparent;
              z-index: 10;
          }
        }
      }
      .progress-section{
          position: absolute;
          bottom: 0;
          left: 0;
          width: 100%;
          .caption-container{
              width: 100%;
              margin-bottom: 10px;
              display: flex;
              align-items: center;
              justify-content: center;
              .caption-text{
                background: rgba(0,0,0,0.2);
                color: #fff;
                padding: 5px;
                max-width: 85%;
                display: inline-block;
                text-align: center;
            }
          }
        }
        .video-controls-container{
            position: absolute;
            top: 0;
            left: 0;
            height: 100%;
            z-index: 100;
            width: 100%;
            color: #fff;
            background-color: rgba(0,0,0,0.2);
            box-sizing: border-box;
            button {
                background: none;
                border: none;
                color: inherit;
                padding: 0;
                cursor: pointer;
                opacity: .85;
                transition: opacity 150ms ease-in-out;
                &.wide-btn {
                  width: 35px;
                  font-size: 14px;
                }
                &:hover {
                    opacity: 1;
                }
            }
            .controls{
              display: flex;
              flex-direction: column;
              justify-content: space-between;
              align-items: center;
              height: 100%;
              width: 100%;
              padding: 0rem .2rem;
              .top{
                  margin-top: 10px;
                  width: 100%;
                  display: flex;
                  justify-content: space-between;
                  align-content: flex-start;
                  .left, .right{
                    display: inline-flex;
                  }
                  button svg{
                    height: 30px;
                    width: auto;
                  }
              }
              .middle{
                  display: inline-flex;
                  justify-content: center;
                  align-items: center;
                  button svg{
                      height: 50px;
                      width: auto;
                  }
              }
              .bottom{
                display: flex;
                justify-content: center;
                align-items: center;
                width: 100%;
                button svg{
                    height: 30px;
                    width: auto;
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
              }
            }
          }
          video {
            width: 100%;
          }
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
    }
  
    .higher{
        position: relative;
        z-index: 1080;
    }
    /* Vue Animation */
    .forwardOne-enter-active,
    .forwardTwo-enter-active,
    .forwardThree-enter-active {
      animation: .1s fade-in;
      opacity: 0;
    }
    .forwardOne-leave-active,
    .forwardTwo-leave-active,
    .forwardThree-leave-active {
      animation: .2s fade-in reverse;
    }
    .forwardTwo-enter-active,
    .forwardTwo-leave-active {
      animation-delay: .15s;
    }

    .forwardThree-enter-active,
    .forwardThree-leave-active {
      animation-delay: .3s;
    }
    /* Fast backward Animation  */
    .backwardOne-enter-active,
    .backwardTwo-enter-active,
    .backwardThree-enter-active {
      animation: .1s fade-in;
      opacity: 0;
    }
    .backwardOne-leave-active,
    .backwardTwo-leave-active,
    .backwardThree-leave-active {
      animation: .2s fade-in reverse;
    }
    .backwardTwo-enter-active,
    .backwardTwo-leave-active {
      animation-delay: .15s;
    }

    .backwardThree-enter-active,
    .backwardThree-leave-active {
      animation-delay: .3s;
    }

    .skipText-enter-active{
      animation: fade-in 0.5s;
    }
    .skipText-leave-active{
      animation: fade-in 0.5s reverse;
    }

     /* Animation  */
    @keyframes fade-in {
      0% {
        opacity: 0;
      }
      100% {
        opacity: 1;
      }
    }
</style>