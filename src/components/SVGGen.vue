<template>
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" :viewBox=viewBox ref="svg">
      <rect x="0" y="0" :width=width :height=height class="background"/>
      <component v-for="(figure, index) in figures"
                 :key="index"
                 :is="figure.type"
                 :data-id="index"
                 class="draggable"
                 :width="figure.width"
                 :height="figure.height"
                 :x="figure.x"
                 :y="figure.y"
                 :cx="figure.x"
                 :cy="figure.y"
                 :r="figure.radius"
                 :stroke="figure.stroke"
                 :stroke-width="figure.strokeWidth"
                 :fill="figure.fill"
                 @click="selectFigure($event)"
      />
    </svg>

    <div v-if="selectedFigure" class="figure-details">
      <form>
        <template v-for="attr in Object.keys(figures[0])">
          <div :key="attr" v-if="figures[selectedFigure][attr]">
            {{attr}}: <input type="text" v-model="figures[selectedFigure][attr]">
          </div>
        </template>
      </form>
    </div>

    <button @click="addFigure('rect')">Add Rect</button>
    <button @click="addFigure('circle')">Add Circle</button>
    <button v-if="selectedFigure" @click="deleteFigure">Delete Figure</button>

  </div>
</template>

<script>
export default {
  name: 'SVGGen',
  props: {
    width: Number,
    height: Number,
    maxFiguresAmt: Number,
    availableFigures: Array[String],
    defaultFigures: Array[Object],
  },
  data() {
    return {
      viewBox: `0 0 ${this.width} ${this.height}`,
      selectedFigure: null,
      figures: [
        {
          type: 'rect',
          x: 4,
          y: 5,
          width: 9,
          height: 9,
          radius: null,
          stroke: 'black',
          strokeWidth: 0.5,
          strokeDashArray: 'none',
          strokeOpacity: 1,
          fill: 'transparent',
          fillOpacity: 0,
        },
        {
          type: 'circle',
          x: 15,
          y: 12,
          width: null,
          height: null,
          radius: 5,
          stroke: 'black',
          strokeWidth: 0.5,
          strokeDashArray: 'none',
          strokeOpacity: 1,
          fill: 'transparent',
          fillOpacity: 0,
        },
      ],
    }
  },
  mounted() {
    var svg = this.$refs.svg;
    svg.addEventListener('mousedown', startDrag);
    svg.addEventListener('mousemove', drag);
    svg.addEventListener('mouseup', endDrag);
    const those = this;

    function getMousePosition(evt) {
      var CTM = svg.getScreenCTM();
      return {
        x: (evt.clientX - CTM.e) / CTM.a,
        y: (evt.clientY - CTM.f) / CTM.d
      };
    }
    var selectedElement, offset;

    function startDrag(evt) {
      if (evt.target.classList.contains('draggable')) {
        selectedElement = evt.target;
        offset = getMousePosition(evt);
        //todo DRY
        const attrX = selectedElement.nodeName === 'rect' ? 'x' : 'cx';
        const attrY = selectedElement.nodeName === 'rect' ? 'y' : 'cy';

        offset.x -= parseFloat(selectedElement.getAttributeNS(null, attrX));
        offset.y -= parseFloat(selectedElement.getAttributeNS(null, attrY));
      }
    }

    function drag(evt) {
      if (selectedElement) {
        //todo DRY
        const attrX = selectedElement.nodeName === 'rect' ? 'x' : 'cx';
        const attrY = selectedElement.nodeName === 'rect' ? 'y' : 'cy';
        var coord = getMousePosition(evt);

        selectedElement.setAttributeNS(null, attrX, coord.x - offset.x);
        selectedElement.setAttributeNS(null, attrY, coord.y - offset.y);

        those.figures[selectedElement.getAttribute('data-id')].x = coord.x - offset.x;
        those.figures[selectedElement.getAttribute('data-id')].y = coord.y - offset.y;
      }
    }

    function endDrag() {
      selectedElement = null;
    }
  },
  methods: {
    selectFigure(e) {
      const figIndex = e.target.getAttribute('data-id');
      this.selectedFigure = figIndex;
    },
    addFigure(type) {
      this.figures.push({
        type,
        x: 4,
        y: 5,
        width: 9,
        height: 9,
        radius: type === 'rect' ? null : 4,
        stroke: 'black',
        strokeWidth: 0.5,
        strokeDashArray: 'none',
        strokeOpacity: 1,
        fill: 'transparent',
        fillOpacity: 0,
      },);
    },
    deleteFigure() {
      this.figures.splice(this.selectedFigure, 1);
    },
  },
}
</script>

<style scoped lang="scss">
.background {
  fill: #eee;
}
.static {
  cursor: not-allowed;
}
.draggable {
  cursor: move;
}
svg {
  margin-bottom: 20px;
}
.figure-details {
  margin-bottom: 20px;
  form div {
    display: inline-block;
    margin: 0 0 10px 20px;
  }
}
button {
  margin-left: 20px;
}
</style>
