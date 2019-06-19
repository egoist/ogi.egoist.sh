<template>
  <div class="container">
    <GitHub slug="egoist/ogi.egoist.sh" />

    <h1 class="site-title">{{ $siteConfig.title }}</h1>

    <div class="form">
      <label for>
        Text
        <input class="input" v-model="output.text">
      </label>

      <label for>
        Text Size
        <input class="input" type="number" v-model="output.textSize">
      </label>

      <label for>
        Logo
        <input class="input" v-model="output.logo.url">
        <button @click="uploadLogo">Upload</button>
        <select v-model="output.logo.width">
          <option value="auto">width</option>
          <option value="100">100</option>
          <option value="150">150</option>
          <option value="200">200</option>
          <option value="250">250</option>
          <option value="300">300</option>
          <option value="350">350</option>
        </select>
        <select v-model="output.logo.height">
          <option value="auto">height</option>
          <option value="100">100</option>
          <option value="150">150</option>
          <option value="200">200</option>
          <option value="250">250</option>
          <option value="300">300</option>
          <option value="350">350</option>
        </select>
      </label>
    </div>

    <div class="output-wrapper">
      <div class="output" ref="output">
        <div class="output-logo" v-if="output.logo.url">
          <img :src="output.logo.url" alt="logo">
        </div>
        <div class="output-text" v-html="renderedText"></div>
      </div>
      <button class="download" @click="download">Download</button>
    </div>

    <component :is="style"></component>

    <div class="introduction">
      <h2>What is this?</h2>
      <p>
        A quick and simple way to create a image as social media preview. This works by using <a href="https://github.com/tsayen/dom-to-image" target="_blank">dom-to-imagea</a> to generate an image from a DOM node, while <a href="https://og-image.now.sh/" target="_blank">og-image.now.sh</a> uses Chrome headless under the hood to take a screenshot of the web page.
      </p>
    </div>

    <footer class="footer">
      Made by <a href="https://github.com/egoist">EGOIST</a>
    </footer>
  </div>
</template>

<script>
import marked from 'marked'
import domToImage from 'dom-to-image'
import GitHub from 'vue-github-badge'
import ogImage from '../assets/images/og.png'

export default {
  components: {
    GitHub
  },

  head() {
    return {
      title: this.$siteConfig.title,
      meta: [
        {
          property: 'og:title',
          content: this.$siteConfig.title
        },
        {
          property: 'og:image',
          content: ogImage
        }
      ]
    }
  },

  data() {
    return {
      output: {
        text: 'Live to see the end of this world',
        textSize: 30,
        logo: {
          url: 'https://cdn.jsdelivr.net/gh/remojansen/logo.ts@master/ts.svg',
          width: 'auto',
          height: 'auto'
        }
      }
    }
  },

  computed: {
    renderedText() {
      return marked(this.output.text)
    },

    style() {
      return {
        render: h => {
          return h('style', {
            domProps: {
              innerHTML: `
              .output-text {
                font-size: ${this.output.textSize}px;
              }
              .output-logo {
                margin-bottom: 50px;
              }
              .output-logo img {
                width: ${
                  this.output.logo.width === 'auto'
                    ? '100px'
                    : `${this.output.logo.width}px`
                };
                height: ${
                  this.output.logo.height === 'auto'
                    ? 'auto'
                    : `${this.output.logo.height}px`
                };
              }
              `
            }
          })
        }
      }
    }
  },

  methods: {
    setLogoURL() {
      this.output.logo.url = window.prompt('Paste the URL')
    },

    uploadLogo() {
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = 'image/jpeg, image/png, image/svg+xml'
      input.onchange = () => {
        this.output.logo.url = URL.createObjectURL(input.files[0])
      }
      input.click()
    },

    async download() {
      const dataUrl = await domToImage.toPng(this.$refs.output, {
        width: 1280,
        height: 640
      })
      const link = document.createElement('a')
      link.download = 'my-image-name.png'
      link.href = dataUrl
      link.click()
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 1280px;
  margin: 10px auto;
  padding: 0 10px;
}

.site-title {
  font-size: 2rem;
  font-weight: 400;
}

.form {
  margin-bottom: 20px;
}

.output {
  width: 100%;
  height: 480px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  flex-direction: column;
  background-color: #fff;
  color: #000;
  background-image: radial-gradient(circle, #D7D7D7, #D7D7D7 1px, #FFF 1px, #FFF);
  background-size: 28px 28px;
}

.output-wrapper {
  position: relative;
  box-shadow: 0 0 1px var(--border-color);
}

/deep/ .output-text>*:first-child {
  margin-top: 0;
}

/deep/ .output-text>*:last-child {
  margin-bottom: 0;
}

.download {
  display: none;
  position: absolute;
  bottom: 10px;
  right: 10px;
}

.output-wrapper:hover > .download {
  display: block;
}

.introduction {
  margin-top: 50px;
}

.introduction h2 {
  font-weight: 400;
  font-size: 1.5rem;
}

.footer {
  margin-top: 50px;
  border-top: 1px solid var(--border-color);
  padding-top: 30px;
}
</style>
