<script setup lang="ts">
// import * as Y from "yjs";
import * as Y from 'yjs'
import { HocuspocusProvider } from '@hocuspocus/provider'
import { reactive, ref } from 'vue'

// Connect it to the backend
const provider = new HocuspocusProvider({
  url: 'ws://127.0.0.1:3000',
  name: 'example-document',
  document: new Y.Doc(),
  onAwarenessUpdate: ({ states }) => {
    states.forEach((state) => {
      setCursorPosition(state.user.mouseX, state.user.mouseY, state.user.name)
    })
  },
})

const randomNum = Math.random()

document.addEventListener('mousemove', (event) => {
  // Share any information you like
  provider.setAwarenessField('user', {
    name: `Sam Bryan ${randomNum}`,
    color: '#ffcc00',
    mouseX: event.clientX,
    mouseY: event.clientY,
  })
})

// Function to set the cursor position
function setCursorPosition(x: number, y: number, name: string) {
  const cursor = document.getElementById('custom-cursor')
  if (!cursor) {
    return
  }
  cursor.style.left = `${x}px`
  cursor.style.top = `${y}px`
  cursor.innerHTML = name
}
const width = window.innerWidth
const height = window.innerHeight

const configKonva = reactive({
  width: width,
  height: height,
})
const dragItemId = ref<number | null>(null)
const list = reactive<{ id: number; x: number; y: number; rotation: number; scale: number }[]>([])

function handleDragstart(e: any) {
  // save drag element:
  dragItemId.value = e.target.id()
  // move current element to the top:
  const item = list.find((i) => i.id === dragItemId.value)
  if (!item) {
    return
  }
  const index = list.indexOf(item)
  list.splice(index, 1)
  list.push(item)
}
function handleDragend() {
  dragItemId.value = null
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  <main>
    <div>
      <v-stage
        ref="stage"
        :config="configKonva"
        @dragstart="handleDragstart"
        @dragend="handleDragend"
      >
        <v-layer ref="layer">
          <v-star
            v-for="item in list"
            :key="item.id"
            :config="{
              x: item.x,
              y: item.y,
              rotation: item.rotation,
              id: item.id,
              numPoints: 5,
              innerRadius: 30,
              outerRadius: 50,
              fill: '#89b717',
              opacity: 0.8,
              draggable: true,
              scaleX: dragItemId === item.id ? item.scale * 1.2 : item.scale,
              scaleY: dragItemId === item.id ? item.scale * 1.2 : item.scale,
              shadowColor: 'black',
              shadowBlur: 10,
              shadowOffsetX: dragItemId === item.id ? 15 : 5,
              shadowOffsetY: dragItemId === item.id ? 15 : 5,
              shadowOpacity: 0.6,
            }"
          ></v-star>
        </v-layer>
      </v-stage>
    </div>

    <div id="custom-cursor" class="custom-cursor"></div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

/* Styles for the custom cursor */
.custom-cursor {
  position: absolute;
  width: 20px;
  height: 20px;
  background-color: red;
  border-radius: 50%;
  pointer-events: none; /* Prevent cursor from interfering with mouse events */
  transform: translate(-50%, -50%); /* Center the cursor on x, y */
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
