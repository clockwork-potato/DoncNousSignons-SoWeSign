<template>
  <div class="min-h-screen bg-slate-100">
    <div class="flex flex-col items-center justify-center h-full">
      <br>
      <h1 class="text-7xl mb-4 font-extrabold text-blue-500 to-blue-900">
        SoWeSign en un clique
      </h1>
      <br>
      <div class="w-full max-w-2xl p-4 bg-white shadow-lg rounded flex flex-col items-center">
        <canvas ref="canvas" width="600" height="400" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing"
          class="border-2 border-gray-300 shadow-lg"></canvas>
        <div class="mt-2 flex flex-wrap gap-2 items-center justify-center">
          <button @click="clearCanvas"
            class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded shadow-lg">Reset moi
            ça</button>
          <button @click="undo"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded shadow-lg">Ctrl Z</button>
          <button @click="generateBookmarkletCode"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded shadow-lg">Générer le
            code</button>
          <button v-if="showBookmarkletCode" @click="copyCode"
            class="bg-yellow-500 hover:bg-yellow-800 text-white font-bold py-2 px-4 rounded shadow-lg">Ctrl C</button>
        </div>
      </div>
      <div class="w-1/2 flex justify-end" v-if="showBookmarkletCode">
        <textarea ref="bookmarkletCode" readonly :value="minifiedCode" rows="10" cols="50"
          class="mt-4 w-full p-2 border border-gray-300 rounded shadow-lg"></textarea>
      </div>
      <div class="p-4 bg-gray-100 rounded w-full max-w-2xl mt-4 shadow-lg">
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
      <br>
      <br>
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
    getCursorPosition(event) {
      const rect = this.$refs.canvas.getBoundingClientRect();
      return {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top
      };
    },
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
      const moveToIndices = this.coordinates.reduce((acc, point, i) => {
        if (point.moveTo) {
          acc.push(i);
        }
        return acc;
      }, []);

      if (moveToIndices.length <= 1) {
        this.clearCanvas();
      } else {
        const index = moveToIndices[moveToIndices.length - 1];
        this.coordinates.splice(index);
        this.redrawLines(this.coordinates);
      }
    },

    clearCanvas() {
      this.coordinates = [];
      this.context.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
    },
    async generateBookmarkletCode() {
      this.drawingData = this.coordinates;
      const drawingDataString = JSON.stringify(this.drawingData);
      const bookmarkletCode = `javascript:(function(){const drawingData = ${drawingDataString};function drawOnCanvas(drawingData) {const canvas = document.querySelector('canvas');if (!canvas) {console.error('bug pas trouvé le canvas');return;}const ctx = canvas.getContext('2d');function redrawLines(ctx, coordinates) {ctx.clearRect(0, 0, canvas.width, canvas.height);for (let i = 0; i < coordinates.length - 1; i++) {const startPoint = coordinates[i];const endPoint = coordinates[i + 1];if (endPoint.moveTo) continue;ctx.beginPath();ctx.moveTo(startPoint.x, startPoint.y);ctx.lineTo(endPoint.x, endPoint.y);ctx.strokeStyle = 'black';ctx.lineWidth = 5;ctx.stroke();ctx.closePath();}}redrawLines(ctx, drawingData);}drawOnCanvas(drawingData);})();`;

      this.minifiedCode = bookmarkletCode;
      this.showBookmarkletCode = true;
    },
  },
};
</script>

