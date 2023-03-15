<template>
  <div class="container min-h-screen flex flex-col items-center justify-center bg-gray-100">
    <h1 class="text-4xl mb-4 font-semibold">Application de dessin Vue.js</h1>
    <p class="mb-4 text-xl text-gray-600 max-w-3xl text-center">
      Une simple application de dessin créée avec Vue.js et stylisée avec Tailwind CSS. Un bookmarklet c'est quoi? <a href="https://www.debugbar.com/fr/bookmarklets-javascript/" class="text-blue-500"
        target="_blank">https://www.debugbar.com/fr/bookmarklets-javascript/</a>
    </p>
    <div class="w-full max-w-2xl p-4 bg-white shadow-md rounded">
      <canvas ref="canvas" width="400" height="400" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing"
        class="border-2 border-gray-300"></canvas>
      <div class="controls mt-4 flex flex-wrap gap-2">
        <button @click="undo"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Annuler</button>
        <button @click="clearCanvas"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Effacer</button>
        <button @click="generateBookmarkletCode"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Générer le code
          Bookmarklet</button>
        <button v-if="showBookmarkletCode" @click="copyCode"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Copier le code</button>
      </div>
      <textarea v-if="showBookmarkletCode" ref="bookmarkletCode" readonly :value="minifiedCode" rows="10" cols="50"
        class="mt-4 w-full p-2 border border-gray-300 rounded"></textarea>
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

