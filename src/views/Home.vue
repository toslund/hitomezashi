<template>
  <div class="main-content">
    <div>
      <hitomezashi-canvas
        :rawXString="rawXString"
        :rawYString="rawYString"
        :colorMode="colorMode"
        :messageLabel="messageLabel"
        :binaryLabel="binaryLabel"
        :charSet="charSet"
        ref="hitomezashi"/>
    </div>
    <div class="settings">
      <div class="settings-title" :style="{width: inputTextWidth}">
        <h1 class="subtitle">Controls</h1>
      </div>
      <div class="text-inputs canvas-control" ref="textInputs">
        <div class="field is-horizontal">
          <div class="field-label is-normal">
            <label class="label">Column Text</label>
          </div>
            <div class="field">
              <p class="control">
              <input class="input" type="text" maxlength="12" id="xstring" name="columnstring" autocomplete="off" v-model="rawXString" v-on:keypress="filterInput($event)">
              </p>
            </div>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-normal">
            <label class="label">Row Text</label>
          </div>
            <div class="field">
              <p class="control">
              <input class="input" type="text" maxlength="12" id="ystring" name="rowstring" autocomplete="off" v-model="rawYString" v-on:keypress="filterInput($event)">
              </p>
            </div>
        </div>
      </div>
      <div class="control canvas-control">
        <label class="radio">
          <input type="radio" name="answer" value="A-Z" v-model="charSet">
          Only A-Z
        </label>
        <label class="radio">
          <input type="radio" name="answer" value="onlyAscii" v-model="charSet">
          ASCII characters
        </label>
      </div>
      <div class="control canvas-control">
        <label class="radio">
          <input type="radio" name="color" value="dark" v-model="colorMode">
          Traditional
        </label>
        <label class="radio">
          <input type="radio" name="color" value="light" v-model="colorMode">
          Black and White
        </label>
      </div>
      <div class="canvas-control">
        <label class="checkbox">
          <input type="checkbox" v-model="messageLabel">
          Show Word Labels
        </label>
      </div>
      <div class="canvas-control">
        <label class="checkbox">
          <input type="checkbox" v-model="binaryLabel">
          Show Binary Labels
        </label>
      </div>
      <div class="canvas-control">
        <button @click="download" class="button">Download</button>
      </div>
    </div>
    <div class="about">
      <h1 class="subtitle">About</h1>
      <p>Hitomezashi stitching is a traditional method of Japanese embroidery that uses a grid pattern on indigo fabric.
        A <a href="https://www.youtube.com/watch?v=JbfhzlMk2eY" target="_blank">video</a> from the YouTube channel Numberphile described how
        to use text to create these patterns. Their method, however, encoded the design based on whether a particular letter
        was a vowel. Unfortunately, this means that each design is essentially a one way algorithm and that a design can't be
        "decoded" to reveal the original message.
      </p>
      <p>
        If we instead create the pattern with the underlying binary data of the message, each design has a unique and "hidden"
        meaning.
      </p>
    </div>
  </div>
</template>

<script>
import HitomezashiCanvas from '@/components/HitomezashiCanvas.vue';

export default {
  name: 'Home',
  props: {
    msg: String,
  },
  components: {
    HitomezashiCanvas,
  },
  data() {
    return {
      messageLabel: 'show',
      binaryLabel: 'show',
      charSet: 'A-Z',
      rawXString: 'Artistic',
      rawYString: 'Message',
      colorMode: 'dark',
      inputTextWidth: '1000px',
    };
  },
  watch: {
  },
  computed: {
  },
  mounted() {
    this.$nextTick(() => {
      let elemWidth = 100;
      if ('textInputs' in this.$refs) {
        elemWidth = this.$refs.textInputs.clientWidth;
      }
      this.inputTextWidth = `${elemWidth}px`;
    });
  },
  methods: {
    download() {
      const canvasImage = this.$refs.hitomezashi.canvasAsImage();

      const xhr = new XMLHttpRequest();
      xhr.responseType = 'blob';
      xhr.onload = function () {
        const a = document.createElement('a');
        a.href = window.URL.createObjectURL(xhr.response);
        a.download = 'image_name.png';
        a.style.display = 'none';
        document.body.appendChild(a);
        a.click();
        a.remove();
      };
      xhr.open('GET', canvasImage); // This is to download the canvas Image
      xhr.send();

      // const newTab = window.open('about:blank', 'your hitomezashi');
      // newTab.document.write(`<img src="${dataURL}" alt="from canvas"/>`);
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
  },
};
</script>

<style scoped>


#around {
  display: flex;
  width: 100%;
  border: 1px dashed orange;
}

#canvas {
  width: 100%;
  max-width: 500px;
}

.main-content {
  display: grid;
  grid-template-columns: 1fr;
  margin: auto;
}

.about {
  padding-top: 2em;
  padding-bottom: 5em;
}

.about > p {
  padding-top: .5em;
  font-size: 1.2rem;
  max-width: 500px;
  margin: auto;
  text-align: left;
}
@media only screen and (min-width: 1100px) {
  .main-content {
    grid-template-columns: 1fr 1fr;
  }
  .about {
    grid-column: 1 / 3;
  }
}
.settings {
  display: grid;
  align-content: center;
}

.settings-title {
  border-bottom: 1px solid #1B294B;
}

.settings-title + .canvas-control {
  padding-top: 1em;
}

.settings > * {
  margin: auto;
}

.canvas-control {
  padding-top: .5em;
  padding-bottom: .5em;
}

input {
  accent-color: #1B294B;
}


</style>
