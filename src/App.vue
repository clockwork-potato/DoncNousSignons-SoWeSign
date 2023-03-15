<template>
  <div class="container min-h-screen flex flex-col items-center justify-center bg-gray-100">
    <h1 class="text-4xl mb-4 font-semibold">SoWeSign en un clique</h1>
    
    </p>
    <div class="w-full max-w-2xl p-4 bg-white shadow-md rounded flex">
      <div class="w-1/2">
        <canvas ref="canvas" width="400" height="400" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing"
          class="border-2 border-gray-300"></canvas>
        <div class="controls mt-4 flex flex-wrap gap-2">
          <button @click="undo" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Reset moi
            ça</button>
          <button @click="clearCanvas" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Ctrl
            Z</button>
          <button @click="generateBookmarkletCode"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Générer le code
            Bookmarklet</button>
          <button v-if="showBookmarkletCode" @click="copyCode"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Ctrl C</button>
        </div>
      </div>
      <div class="w-1/2" v-if="showBookmarkletCode">
        <textarea ref="bookmarkletCode" readonly :value="minifiedCode" rows="10" cols="50"
          class="mt-4 w-full p-2 border border-gray-300 rounded"></textarea>
      </div>
    </div>
    <div class="p-4 bg-gray-100 rounded w-full max-w-2xl mt-4">
      <h2 class="text-lg font-semibold mb-2">Qu'est-ce qu'un bookmarklet ?</h2>
      <p class="text-gray-600 text-sm mb-4">Un bookmarklet est un petit programme JavaScript qui peut être enregistré
        dans les favoris ou les signets de votre navigateur. Lorsqu'il est cliqué, il exécute le code JavaScript contenu
        dans le bookmarklet sur la page web actuelle. Les bookmarklets peuvent être utilisés pour effectuer des tâches
        spécifiques sur des pages web, comme ajouter un bouton de partage de médias sociaux, afficher des informations
        supplémentaires sur une page ou même dessiner sur une page web, comme dans l'exemple de code Vue.js que vous
        avez fourni.</p>
      <p class="text-gray-600 text-sm">Le code JavaScript contenu dans un bookmarklet doit être court et efficace pour
        ne pas ralentir la page web et pour s'assurer qu'il fonctionne sur toutes les pages web. C'est pourquoi il est
        important de minifier le code avant de l'insérer dans un bookmarklet.</p>
    </div>

  </div>
</div></template>




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
    copyCode() {
      navigator.clipboard.writeText(this.minifiedCode);
    },
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

