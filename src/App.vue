<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100">
    <div class="w-full max-w-2xl p-4 bg-white shadow-md rounded">
      <canvas ref="canvas" width="600" height="400" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing" class="border-2 border-gray-300"></canvas>
      <div class="mt-4 flex flex-wrap gap-2">
        <button @click="undo" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Ctrl + z</button>
        <button @click="clearCanvas" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Reset le tout</button>
        <button @click="generateBookmarkletCode" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Genere moi le code</button>
        <button v-if="showBookmarkletCode" @click="copyCode" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Ctrl +c</button>
      </div>
      <textarea v-if="showBookmarkletCode" ref="bookmarkletCode" readonly :value="minifiedCode" rows="10" cols="50" class="mt-4 w-full p-2 border border-gray-300 rounded"></textarea>
    </div>
  </div>
</template>



<script>


export default {
  data() {
    return {
      drawing: false,
      context: null,
      coordinates: [],
      drawingData: [],
      showBookmarkletCode: false,
      minifiedCode: "",
    };
  },
  mounted() {
  this.context = this.$refs.canvas.getContext("2d");
},


  methods: {
    startDrawing(event) {
  this.drawing = true;
  this.coordinates.push({ x: event.offsetX, y: event.offsetY, moveTo: true });
},

draw(event) {
  if (!this.drawing) return;
  const x = event.offsetX;
  const y = event.offsetY;
  this.coordinates.push({ x, y });
  this.redrawLines(this.coordinates);
},


    stopDrawing() {
      this.drawing = false;
    },
    redrawLines(coordinates) {
      this.context.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);

      for (let i = 0; i < coordinates.length - 1; i++) {
        const startPoint = coordinates[i];
        const endPoint = coordinates[i + 1];

        if (endPoint.moveTo) continue;

        this.context.beginPath();
        this.context.moveTo(startPoint.x, startPoint.y);
        this.context.lineTo(endPoint.x, endPoint.y);
        this.context.strokeStyle = "black";
        this.context.lineWidth = 5;
        this.context.stroke();
        this.context.closePath();
      }
    },
    undo() {
      const index = this.coordinates
        .map((point, i) => (point.moveTo ? i : -1))
        .filter((i) => i !== -1)
        .slice(-2)[0];
      if (index > 0) {
        this.coordinates.splice(index);
        this.redrawLines(this.coordinates);
      } else {
        this.clearCanvas();
      }
    },
    clearCanvas() {
      this.coordinates = [];
      this.context.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
    },


    async generateBookmarkletCode() {
      this.drawingData = this.coordinates;
      const drawingDataString = JSON.stringify(this.drawingData);
      const bookmarkletCode = `
    javascript:(function(){
      const drawingData = ${drawingDataString};
      function drawOnCanvas(drawingData) {
        const canvas = document.querySelector('canvas');
        if (!canvas) {
            console.error('bug pas trouvé le canvas');
            return;
        }
        const ctx = canvas.getContext('2d');
        
        function redrawLines(ctx, coordinates) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            for (let i = 0; i < coordinates.length - 1; i++) {
                const startPoint = coordinates[i];
                const endPoint = coordinates[i + 1];

                if (endPoint.moveTo) continue;

                ctx.beginPath();
                ctx.moveTo(startPoint.x, startPoint.y);
                ctx.lineTo(endPoint.x, endPoint.y);
                ctx.strokeStyle = 'black';
                ctx.lineWidth = 5;
                ctx.stroke();
                ctx.closePath();
            }
        }

        redrawLines(ctx, drawingData);
    }

    drawOnCanvas(drawingData);
})();`;

      this.minifiedCode = bookmarkletCode;
      this.showBookmarkletCode = true;
    },
  },
};
</script>

