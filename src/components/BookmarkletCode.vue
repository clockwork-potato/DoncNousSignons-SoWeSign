<template>
    <div>
      <h3>Bookmarklet Code:</h3>
      <textarea ref="bookmarkletCode" readonly :value="minifiedCode" rows="10" cols="50"></textarea>
      <button @click="copyCode">Copy Code</button>
    </div>
  </template>
  
  <script>
  import { minify } from "terser";
  
  export default {
    props: {
      drawingData: {
        type: Array,
        required: true,
      },
    },
    data() {
      return {
        minifiedCode: "",
      };
    },
    async mounted() {
      const bookmarkletCode = `
        javascript:(function(){
          const drawingData = ${JSON.stringify(this.drawingData)};
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
  
      const result = await minify(bookmarkletCode);
      this.minifiedCode = result.code;
    },
    methods: {
      copyCode() {
        this.$refs.bookmarkletCode.select();
        document.execCommand("copy");
        alert("Code copied to clipboard!");
      },
    },
  };
  </script>
  