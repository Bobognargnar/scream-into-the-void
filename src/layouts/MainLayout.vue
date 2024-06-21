<template>
  <q-layout view="lHh Lpr lFf">

    <q-page-container>
      
  <!-- <button v-on:click="show = !show">
    Toggle
  </button> -->
      <div style="position: relative; z-index: 2;" ref="svgContainer" @mousedown="startDrag">
      <svg width="100%" height="100%" viewBox="0 0 200 200" style="z-index: 1;">
        

        <circle 
          ref="draggableCircle"
          :cx="position.x"
          :cy="position.y"
          r="45%"
          stroke="black"
          stroke-width="3"
          fill="black"
          class="pulsing-image"
          
        >
        <!-- <animate attributeName="r" begin="0s" dur="1s" repeatCount="indefinite" from="45%" to="50%"/>   -->
        </circle>
        <text 
          id="dragme"
          :x="position.x" 
          :y="position.y+70" 
          text-anchor="middle" 
          dominant-baseline="middle" 
          fill="white"
          class="pulsing-text chivo-mono-font"
        >
        DRAG THE VOID
      </text>
      <foreignObject 
        :x="position.x-15" 
        :y="position.y+60" 
        width="80%" 
        height="200"
        style="transform: translate(-25%, -20%);">
        <transition name="fade">
        <div v-if="show">
          <textarea 
          spellcheck="false"
          class = "custom-textarea"
          v-model="message" placeholder="Scream into me"
          ></textarea>
        </div>
      </transition>
      </foreignObject>
    </svg>
    
  </div>  
    </q-page-container>
  </q-layout>
</template>

<script>
import { reactive, computed } from 'vue'
import dynamics from 'dynamics.js'
import { useTimeout } from 'quasar'
import { ref } from 'vue';

//document.documentElement.style.overflow = 'hidden'

export default {
  data() {
    return {
      lockDragging: false,
      start_position: { x: 100, y: -60 },
      position: { x: 0, y: 0 },
      isDragging: false,
      offset: { x: 0, y: 0 },
      start: { x: 0, y: 0 },
      c: reactive({ x: 100, y: -20 }),
      message: '',
      timeoutId: null,
      show: true,
      registerTimeout: useTimeout()
    };
  },
  watch: {
    message(newVal, oldVal) {
      
      this.show = true
      // When user starts writing, a timer starts that "disappears" the text
      this.startTextDissolve()

    }
  },
  methods: {
    startDrag(event) {
      this.isDragging = true;
      // Calculate the offset between the mouse click and the center of the circle
      const circle = this.$refs.draggableCircle;
      const circleBox = circle.getBoundingClientRect();
      this.offset = {
        x: event.clientX - circleBox.x - circleBox.width / 2,
        y: event.clientY - circleBox.y - circleBox.height / 2,
      };
      
      this.start.x = event.pageX
      this.start.y = event.pageY
      
      console.log(window.innerWidth)
      console.log(event.clientX)
      console.log(`Offset`)
      console.log(this.offset)
      // Add event listeners for mouse move and up
      if (!this.lockDragging){
      document.addEventListener('mousemove', this.onDrag);
      document.addEventListener('mouseup', this.stopDrag);}
    },
    onDrag(event) {
      if (this.isDragging) {
        // Only draggable down
        if ((this.start.y - event.clientY)<0) {
          this.position = {
            x: this.position.x,
            y: this.position.y - (this.start.y - event.clientY)*0.15
          };
          this.start.y = event.clientY
        }
        console.log(this.position)
        if (this.position.y >= this.start_position.y + 40) {
          this.lockDragging = true
          this.stopDrag()
        }
      }
    },
    stopDrag() {
      this.isDragging = false;
      // Remove event listeners for mouse move and up
      document.removeEventListener('mousemove', this.onDrag);
      document.removeEventListener('mouseup', this.stopDrag);
      if (!this.lockDragging) {
        dynamics.animate(
          this.position,
          this.start_position,
          { type: dynamics.spring, duration: 5000, friction: 250 }
        )
      } else {
        this.initializeVoid()
      }
    },
    updatePosition() {
      this.position.y = this.start_position.y
      this.position.x = this.start_position.x
    },
    initializeVoid() {
      const element = document.getElementById("dragme");
      element.classList.add("dissolve-text");
      document.getElementsByClassName('custom-textarea')[0].style.display = "block";
      document.getElementsByClassName('custom-textarea')[0].classList.add('appear-text')
      
    },
    startTextDissolve () {
      if (this.timeoutId) {
        clearTimeout(this.timeoutId);
      }
      this.timeoutId = setTimeout(() => {
        this.show = false;
      }, 500);
    },
  },
  mounted() {
    this.updatePosition();
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Chivo+Mono:ital,wght@0,100..900;1,100..900&display=swap');

svg {
  /*border: 1px solid #ccc;*/
}

/* input {
  font-size: 16px;
  line-height: 1.5;
} */

.fade-enter-active, .fade-leave-active {
  transition: opacity 5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

.pulsing-text {
  user-select: none;
  animation: pulse 2s infinite;
  transform-origin:top;
}

.pulsing-image {
  user-select: none;
  animation: pulse-image 2s infinite;
  transform-origin:top;
}


.chivo-mono-font {
  font-size: 80%;
  font-family: "Chivo Mono", monospace;
  font-optical-sizing: auto;
  font-weight: 300;
  font-style: normal;
}

@keyframes pulse {
  0% {
    opacity:1;
  }
  50% {
    opacity:0.3;
  }
  100% {
    opacity:1;
  }
}
@keyframes pulse-image {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.dissolve-text {
  animation: dissolve 1s forwards;
}

@keyframes dissolve {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

.appear-text {
  animation: appear 1s forwards;
}

@keyframes appear {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}


.custom-textarea {
  transition: opacity 5s ease-in-out;
  width: 80%; /* Adjust the width as needed */
  height: 35px; /* Adjust the height as needed */
  caret-color: auto;
  font-size: 30%;
  background-color: transparent;
  color: white;
  outline: none;
  resize: none;
  display: auto; 
  border: none;
}

.dissolve-enter-active, .dissolve-leave-active {
  transition: opacity 5s ease-in-out;
}

.dissolve-enter, .dissolve-leave-to /* .dissolve-leave-active in <2.1.8 */ {
  opacity: 0;
}
</style>
