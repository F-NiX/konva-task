<template>
  <v-stage
    ref="stage"
    :config="stageSize"
    @mousemove="handleMouseMove"
    @mouseUp="handleMouseUp"
  >
    <v-layer ref="layer">
      <v-text
        ref="text"
        :config="{
          x: 10,
          y: 10,
          fontSize: 20,
          text: 'Draw a cube',
          fill: 'black',
        }"
      />
      <v-rect
        v-for="(rect, index) in rects"
        :key="index"
        :config="{
          x: Math.min(rect.startPointX, rect.startPointX + rect.width),
          y: Math.min(rect.startPointY, rect.startPointY + rect.height),
          width: Math.abs(rect.width),
          height: Math.abs(rect.height),
          fill: 'rgb(0,0,0,0)',
          stroke: 'black',
          strokeWidth: 3,
        }"
      />
      <v-line
        v-for="(line, index) in lines"
        :key="index + 1000"
        :config="{
          x: line.x,
          y: line.y,
          points: [0, 0, line.dx, line.dy],
          stroke: 'black',
          strokeWidth: 3,
        }
      "/>
    </v-layer>
  </v-stage>
</template>

<script>
const width = window.innerWidth
const height = window.innerHeight

// In TypeScript, enum can be used
const ClickStates = {
  FREE: 0, RECT: 1, DEPTH: 2, DONE: 3
}

export default {
  data: () => ({
    stageSize: { width, height },
    // Edges
    lines: [],
    // Faces
    rects: [],
    /*
    0 -> FREE:  state (no drawing)
    1 -> RECT:  drawing first face
    2 -> DEPTH: make it 3D :))
    3 -> DONE:  jump to FREE
    */
    clickState: 0,
  }),
  methods: {
    /**
     * A listener for clicking event on stage
     * Summary: handles drawing states and appending new shape objects, whenever while clicking action dispatches
     * Description: It's is in the function body
     */
    handleMouseUp() {
      /*
      By each click, we change phase
      This goes from:
        FREE state -> RECT drawing state
        RECT drawing state -> Adding DEPTH to make it cube box,
        DEPTH mode -> DONE and reset to -> FREE
      */
      this.clickState++
      if (this.clickState === ClickStates.DONE)
        this.clickState = ClickStates.FREE
      
      /*
      In RECT mode, we simply draw a rect, similar to the example
      size changing is controlled in handleMouseMove method
      */
      if (this.clickState === ClickStates.RECT) {
        const pos = this.$refs.stage.getNode().getPointerPosition()
        this.setRects([
          ...this.rects,
          { startPointX: pos.x, startPointY: pos.y, width: 0, height: 0 },
        ])
      }

      /*
      In DEPTH mode, we add another rect side, and four connecting edges
      it's obvious that edge vertices, each must be on the related rect vertex
      */
      else if (this.clickState === ClickStates.DEPTH) {
        const pos = this.$refs.stage.getNode().getPointerPosition()
        const { width, height } = this.rects[this.rects.length - 1]
        this.setRects([
          ...this.rects,
          { startPointX: pos.x - width, startPointY: pos.y - height, width, height },
        ])
        this.setLines([
          ...this.lines,
          { x: pos.x, y: pos.y, dx: 0, dy: 0 },
          { x: pos.x - width, y: pos.y, dx: 0, dy: 0 },
          { x: pos.x, y: pos.y - height, dx: 0, dy: 0 },
          { x: pos.x - width, y: pos.y - height, dx: 0, dy: 0 },
        ])
      }
    },
    /**
     * in typescript we could define Rect type
     * which has startPoints and Size (width, height)
     * @param {Array of Rects} element replaces the reactive rects object to notify changes
     */
    setRects(element) {
      this.rects = element
    },
    /**
     * Just the same as previous method (setRects) for edges
     * @param {Array of Lines} element
     */
    setLines(element) {
      this.lines = element
    },
    /**
     * Listener for editing cube size by moving mouse
     * in RECT drawing phases: This function controls width and height of the first face of box
     * in DEPTH state: Handles the position of next fixed-size face, and connecting edges between
     */
    handleMouseMove(_event) {
      // No drawing, Skipping
      if (this.clickState === ClickStates.FREE)
        return
      if (this.clickState === ClickStates.RECT) {
        const point = this.$refs.stage.getNode().getPointerPosition()
        // Handles drawing rectangle part
        let curRec = this.rects[this.rects.length - 1]
        curRec.width = point.x - curRec.startPointX
        curRec.height = point.y - curRec.startPointY
      }
      else if (this.clickState === ClickStates.DEPTH) {
        /*
        Adding depth in order to convert rect to cube
        which includes another rectangle and four lines
        */
        const point = this.$refs.stage.getNode().getPointerPosition()
        // 3rd dimension face
        let curRec = this.rects[this.rects.length - 1]
        // Vertex connecting line
        let edge1 = this.lines[this.lines.length - 4]
        let edge2 = this.lines[this.lines.length - 3]
        let edge3 = this.lines[this.lines.length - 2]
        let edge4 = this.lines[this.lines.length - 1]
        // Edge deform
        edge1.dx = point.x - edge1.x
        edge1.dy = point.y - edge1.y
        edge2.dx = point.x - edge2.x - curRec.width
        edge2.dy = point.y - edge2.y
        edge3.dx = point.x - edge3.x
        edge3.dy = point.y - edge3.y - curRec.height
        edge4.dx = point.x - edge4.x - curRec.width
        edge4.dy = point.y - edge4.y - curRec.height
        // Face movement
        curRec.startPointX = point.x - curRec.width
        curRec.startPointY = point.y - curRec.height
      }
    },
  },
}
</script>

<style>
body {
  margin: 0;
}
</style>
