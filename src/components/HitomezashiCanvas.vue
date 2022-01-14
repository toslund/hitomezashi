<template>
  <div class="about">
    <div>
      <canvas @mousemove="handleHover($event)" @mousedown="handleClick($event)" id="canvas" ref="canvas" :width="canvasWidth" :height="canvasHeight" ></canvas>
    </div>
    <!-- <div>
      {{ xString }}
    </div>
    <div>
      {{ yString }}
    </div>
    <div>
      {{ rowColBits }}
    </div>
    <div>
      {{ hover }}
    </div> -->
  </div>
</template>

<script>
import _ from 'lodash';

export default {
  name: 'Home',
  props: {
    rawXString: {
      type: String,
      default: 'Artistic',
    },
    rawYString: {
      type: String,
      default: 'Message',
    },
    colorMode: {
      type: String,
      default: 'dark',
    },
    messageLabel: {
      type: String,
      default: 'show',
    },
    binaryLabel: {
      type: String,
      default: 'show',
    },
    charSet: {
      type: String,
      default: 'A-Z',
    },
  },
  data() {
    return {
      // messageLabel: 'show',
      // binaryLabel: 'show',
      // charSet: 'A-Z',
      dpi: 300,
      canvasWidth: 1000,
      canvasHeight: 1000,
      // rawXString: 'Artistic',
      // rawYString: 'Message',
      hover: { x: 0, y: 0 },
      canvas: { width: 0, height: 0 },
      ctx: null,
      canvasMarginInch: 0.3,
      letterSizePt: 6,
      binarySizePt: 3.5,
      binaryOffsetPt: 3,
      grid: [],
      // colorMode: 'dark',
    };
  },
  watch: {
    rowColBits() {
      console.log('watcher triggered');
      this.render();
    },
    charSet() {
      this.cleanseInputs();
    },
    binaryLabel() {
      this.render();
    },
    messageLabel() {
      this.render();
    },
    darkMode() {
      this.render();
    },
  },
  computed: {
    darkMode() {
      return this.colorMode === 'dark';
    },
    backgroundColor() {
      if (this.darkMode) {
        return '#1B294B';
      }
      return '#FFFFFF';
    },
    stitchColor() {
      if (this.darkMode) {
        return '#FFFFFF';
      }
      return '#080808';
    },
    letterColor() {
      if (this.darkMode) {
        return '#FFFFFF';
      }
      return '#080808';
    },
    binaryColor() {
      if (this.darkMode) {
        return '#FFFFFF';
      }
      return '#080808';
    },
    bitSize() {
      if (this.charSet === 'A-Z') {
        return 5;
      }
      return 7;
    },
    scaleFactor() {
      if (this.shortside === 'columns') {
        return this.canvasHeight / this.computedHeight;
      }
      if (this.shortside === 'rows') {
        return this.canvasWidth / this.computedWidth;
      }
      return 1;
    },
    translateX() {
      if (this.shortside === 'columns') {
        return (this.canvasWidth - ((this.columnSpaces * this.columnSpace) + (2 * this.canvasMargin))) / 2;
      } 
      return 0;
    },
    translateY() {
      if (this.shortside === 'rows') {
        return (this.canvasHeight - ((this.rowSpaces * this.columnSpace) + (2 * this.canvasMargin))) / 2;
      } 
      return 0;
    },
    computedHeight() {
      return (this.columnSpace * this.gridSpaces) + (2 * this.canvasMargin);
    },
    computedWidth() {
      return (this.columnSpace * this.gridSpaces) + (2 * this.canvasMargin);
    },
    columnSpaces() {
      return this.rowColBits.columnBits.length - 1;
    },
    rowSpaces() {
      return this.rowColBits.rowBits.length - 1;
    },
    gridSpaces() {
      return Math.max(this.columnSpaces, this.rowSpaces);
    },
    columnSpace() {
      return (this.canvas.width - (2 * this.canvasMargin)) / (this.gridSpaces);
    },
    shortside() {
      if (this.rawXString.length < this.rawYString.length) {
        return 'columns';
      }
      if (this.rawYString.length < this.rawXString.length) {
        return 'rows';
      }
      return null;
    },
    binaryOffsetPixels() {
      return Math.floor(this.dpi * (this.binaryOffsetPt / 72));
    },
    binarySizePixels() {
      return Math.floor(this.dpi * (this.binarySizePt / 72));
    },
    letterSizePixels() {
      return Math.floor(this.dpi * (this.letterSizePt / 72));
    },
    canvasMargin() {
      return this.dpi * this.canvasMarginInch;
    },
    columnBits() {
      const xArray = [];
      for (let i = 0; i < this.xString.length; i += 1) {
        xArray.push(...this.charToArray(this.xString[i]));
      }
      return xArray;
    },
    xString() {
      let str = null;
      if (this.charSet === 'A-Z') {
        str = this.rawXString.toUpperCase();
      } else if (this.charSet === 'onlyAscii') {
        str = this.rawXString;
      }
      return this.cleanseInput(str);
    },
    yString() {
      let str = null;
      if (this.charSet === 'A-Z') {
        str = this.rawYString.toUpperCase();
      } else if (this.charSet === 'onlyAscii') {
        str = this.rawYString;
      }
      return this.cleanseInput(str);
    },
    rowBits() {
      const yArray = [];
      for (let i = 0; i < this.yString.length; i += 1) {
        yArray.push(...this.charToArray(this.yString[i]));
      }
      return yArray;
    },
    rowColBits() {
      return { columnBits: this.columnBits, rowBits: this.rowBits };
    },
  },
  mounted() {
    this.initCanvas();
    this.render();
  },
  methods: {
    canvasAsImage() {
      const dataURL = this.canvas.toDataURL('image/png');
      return dataURL;
    },
    filterInput(e) {
      console.log('filtering input');
      const char = String.fromCharCode(e.keyCode); // Get the character
      if (this.isCharAllowed(char)) {
        return true;
      }
      e.preventDefault(); // If not match, don't add to input text
      return false;
    },
    cleanseInputs() {
      this.rawXString = this.cleanseInput(this.rawXString);
      this.rawYString = this.cleanseInput(this.rawYString); 
    },
    cleanseInput(inputStr) {
      const chars = inputStr.split('');
      const goodChars = [];
      for (let i = 0; i < chars.length; i += 1) {
        if (this.isCharAllowed(chars[i])) {
          goodChars.push(chars[i]);
        }
      }
      return goodChars.join('');
    },
    isCharAllowed(char) {
      if (this.charSet === 'A-Z') {
        const n = new TextEncoder().encode(char.toUpperCase())[0];
        console.log('CHAR CODE');
        console.log(char);
        console.log(n);
        if (n > 64 && n < 91) { 
          return true;
        }
        return false;
      }
      if (this.charSet === 'onlyAscii') {
        const n = new TextEncoder().encode(char)[0];
        console.log('CHAR CODE');
        console.log(char);
        console.log(n);
        if (n < 127) {
          return true;
        }
        console.log(`UNKNOWN NON ASCII CHAR: ${char}`);
        return false;
      }
      return false; // defualt case only if incorrect charset
    },
    initCanvas() {
      console.log('init canvas');
      this.canvas = this.$refs.canvas;
      if (this.canvas.getContext) {
        this.ctx = this.canvas.getContext('2d');
      }   
      this.render();
    },
    handleClick(event) {
      console.log(event);
    },
    handleHover(event) {
      this.hover.x = event.clientX - this.canvas.getBoundingClientRect().left;
      this.hover.y = event.layerY;
    },
    initGrid() {
      this.grid = [];
      for (let idxRow = 0; idxRow < this.rowColBits.rowBits.length; idxRow += 1) {
        const y = this.canvasMargin + (idxRow * this.columnSpace);
        // console.log(`CREATING ROW: ${idxRow} WITH Y: ${y}`);
        const row = [];
        for (let idxColumn = 0; idxColumn < this.rowColBits.columnBits.length; idxColumn += 1) {
          const x = this.canvasMargin + (idxColumn * this.columnSpace);
          row.push({ x, y });
        }
        // console.log(`CREATED ROW: ${row}`);
        this.grid.push(row);
      }
    },
    drawPoints() {
      this.ctx.fillStyle = this.stitchColor;
      for (let iX = 0; iX < this.rowColBits.columnBits.length; iX += 1) {
        const x = this.canvasMargin + (iX * this.columnSpace);
        for (let iY = 0; iY < this.rowColBits.rowBits.length; iY += 1) {
          const y = this.canvasMargin + (iY * this.columnSpace);
          this.ctx.beginPath();
          this.ctx.arc(x, y, 2, 0, 2 * Math.PI);
          this.ctx.fill();
          this.ctx.closePath();
        }
      }
    },
    connectPoints(firstPoint, secondPoint) {
      this.ctx.strokeStyle = this.stitchColor;
      this.ctx.beginPath();
      this.ctx.moveTo(firstPoint.x, firstPoint.y);
      this.ctx.lineTo(secondPoint.x, secondPoint.y);
      this.ctx.closePath();
      this.ctx.stroke();
    },
    drawLines() {
      // vertical lines
      for (let i = 0; i < this.rowColBits.columnBits.length; i += 1) {
        const idxColumn = this.rowColBits.columnBits.length - 1 - i;
        // console.log(`COLUMN: ${idxColumn}`);
        for (let idxRow = 0; idxRow < this.rowColBits.rowBits.length; idxRow += 1) {
          // console.log(`ROW: ${idxRow}`);
          // console.log(`x: ${this.grid[idxRow][idxColumn].x}, y: ${this.grid[idxRow][idxColumn].y}`);
          if (idxRow !== this.rowColBits.rowBits.length - 1 && idxRow % 2 !== this.rowColBits.columnBits[idxColumn]) {
            // draw down
            const firstPoint = this.grid[idxRow][idxColumn];
            const secondPoint = this.grid[idxRow + 1][idxColumn];
            this.connectPoints(firstPoint, secondPoint);
          }
          if (idxColumn !== 0 && i % 2 !== this.rowColBits.rowBits[idxRow]) {
            // draw left
            const firstPoint = this.grid[idxRow][idxColumn];
            const secondPoint = this.grid[idxRow][idxColumn - 1];
            this.connectPoints(firstPoint, secondPoint);
          }
        }
      }
      // horizontal lines
      // for (let iY = 0; iY < this.rowColBits.rowBits.length; iY += 1) {
      //   for (let iX = 1; iX < this.rowColBits.columnBits.length; iX += 1) {
      //     if (iX % 2 === this.rowColBits.rowBits[iY]) {
      //       this.ctx.beginPath();
      //       this.ctx.moveTo(this.grid[this.rowColBits.columnBits.length - (iX)][iY].x, this.grid[this.rowColBits.columnBits.length - (iX)][iY].y);
      //       this.ctx.lineTo(this.grid[this.rowColBits.columnBits.length - (iX + 1)][iY].x, this.grid[this.rowColBits.columnBits.length - (iX + 1)][iY].y);
      //       this.ctx.closePath();
      //       this.ctx.stroke();
      //     }
      //   }
      // }
    },
    drawLetter(letter, axis, n) {
      this.ctx.fillStyle = this.letterColor;
      this.ctx.font = `${this.letterSizePixels}px Arial`;
      const letterSize = this.ctx.measureText(letter);
      const letterHeight = letterSize.actualBoundingBoxAscent + letterSize.actualBoundingBoxDescent;
      let x = 0;
      let y = 0;
      const firstBit = (n * this.bitSize);
      // console.log(`first bit: ${firstBit}`);
      const secondBit = ((n + 1) * this.bitSize) - 1;
      // console.log(`second bit: ${secondBit}`);
      const centeredBit = (secondBit + firstBit) / 2;
      // console.log(`centered bit: ${centeredBit}`);
      const centeredPixel = (centeredBit * this.columnSpace) + this.canvasMargin;
      if (axis === 'column') {
        x = centeredPixel - letterSize.width / 2;
        y = letterHeight + this.canvasMargin / 4;
      } else if (axis === 'row') {
        x = this.canvas.width - letterSize.width - this.translateX - (this.canvasMargin / 4); // (this.canvasMargin / 4)
        y = centeredPixel + letterHeight / 2;
      }
      // console.log(`drawing letter: ${letter} at x: ${x} and y: ${y}`);
      this.ctx.fillText(letter, x, y);
    },
    drawBit(letter, axis, n) {
      this.ctx.font = `${this.binarySizePixels}px Arial`;
      const formattedLetter = letter === 1 ? 'l' : '0';
      const letterSize = this.ctx.measureText(formattedLetter);
      const letterHeight = letterSize.actualBoundingBoxAscent + letterSize.actualBoundingBoxDescent;
      let x = 0;
      let y = 0;
      const centeredPixel = (n * this.columnSpace) + this.canvasMargin;
      if (axis === 'column') {
        x = centeredPixel - (letterSize.width / 2);
        y = this.grid[0][0].y - this.binaryOffsetPixels;
      } else if (axis === 'row') {
        x = this.grid[0][this.grid[0].length - 1].x + this.binaryOffsetPixels + 10 - (letterSize.width / 2);
        y = centeredPixel + letterHeight / 2;
      }
      // console.log(`drawing letter: ${letter} at x: ${x} and y: ${y}`);
      // because the 1 looks serif, use L
      this.ctx.fillText(formattedLetter, x, y);
    },
    debounceRender: _.debounce(function () {
      this.render();
    }, 100),
    render() {
      console.log('rendering');
      this.ctx.resetTransform();
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.ctx.fillStyle = this.backgroundColor;
      this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
      // ctx.setTransform(1, 0, 0, 1, 0, 0);
      this.ctx.translate(this.translateX, this.translateY);
      this.ctx.scale(this.scaleFactor, this.scaleFactor);
      this.ctx.fillStyle = 'rgb(200, 0, 0)';
      this.initGrid();
      this.drawPoints();
      this.drawLines();
      if (this.messageLabel) {
        this.xString.split('').forEach((letter, index) => {
          this.drawLetter(letter, 'column', index);
        });
        this.yString.split('').forEach((letter, index) => {
          this.drawLetter(letter, 'row', index);
        });
      }
      if (this.binaryLabel) {
        this.rowColBits.columnBits.forEach((letter, index) => {
          this.drawBit(letter, 'column', index);
        });
        this.rowColBits.rowBits.forEach((letter, index) => {
          this.drawBit(letter, 'row', index);
        });
      }
    },
    arrayToChar(intArray) {
      const simpleInt = parseInt(intArray.map((r) => (r ? '1' : '0').join('')), 2);
      const charCode = (1 << 6) | simpleInt; // eslint-disable-line no-bitwise
      String.fromCharCode(charCode);
    },
    charToArray(char) {
      const n = new TextEncoder().encode(char)[0];
      let charArray = [];
      if (this.bitSize === 5) {
        const fiveBitN = n ^ 64; // eslint-disable-line no-bitwise
        charArray = fiveBitN.toString(2).split('').map((r) => (r === '1' ? 1 : 0));
        while (charArray.length < 5) {
          charArray.unshift(0);
        }
      } else {
        charArray = n.toString(2).split('').map((r) => (r === '1' ? 1 : 0));
      }
      return charArray;
    },
  },
};
</script>

<style scoped>


#around{
  display: flex;
  width: 100%;
  border: 1px dashed orange;
}

#canvas{
  width: 100%;
  max-width: 500px;
}


</style>
