<template>
  <div @pointerover.once="warmup">
    <div class="wrapper" :style="wrapperStyle">
      <div v-if="!isClicked" @click="click" :style="thumbStyle" class="thumb">
        <button aria-label="play">
          <svg height="100%" version="1.1" viewBox="0 0 68 48" width="100%">
            <path
              class="ytp-large-play-button-bg"
              d="M66.52,7.74c-0.78-2.93-2.49-5.41-5.42-6.19C55.79,.13,34,0,34,0S12.21,.13,6.9,1.55 C3.97,2.33,2.27,4.81,1.48,7.74C0.06,13.05,0,24,0,24s0.06,10.95,1.48,16.26c0.78,2.93,2.49,5.41,5.42,6.19 C12.21,47.87,34,48,34,48s21.79-0.13,27.1-1.55c2.93-0.78,4.64-3.26,5.42-6.19C67.94,34.95,68,24,68,24S67.94,13.05,66.52,7.74z"
              fill="#212121"
              fill-opacity="0.8"
            ></path>
            <path d="M 45,24 27,14 27,34" fill="#fff"></path>
          </svg>
        </button>
      </div>
      <iframe
        v-else
        :src="src"
        frameborder="0"
        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
      ></iframe>
    </div>
  </div>
</template>

<script>
const embedUrlBase = "https://www.youtube.com/embed";
const thumbUrlBase = "https://i.ytimg.com/vi";

export default {
  name: "YouTube",
  props: {
    aspect: {
      type: Number,
      default: 9 / 16
    },
    vid: {
      type: String,
      required: true
    },
    thumbQuality: {
      type: String,
      default: "sd" /* mq, hq, sd - ascending order of resolution */
    }
  },
  data() {
    return {
      isClicked: false,
      hasPreconnected: false
    };
  },
  computed: {
    src() {
      return [embedUrlBase, this.vid].join("/") + "?autoplay=1";
    },
    thumbUrl() {
      return [thumbUrlBase, this.vid, `${this.thumbQuality}default.jpg`].join(
        "/"
      );
    },
    wrapperStyle() {
      return {
        "padding-bottom": `${this.aspect * 100}%`
      };
    },
    thumbStyle() {
      return {
        "background-image": `url(${this.thumbUrl})`
      };
    }
  },
  methods: {
    addPrefetch(kind, url, as) {
      const linkElem = document.createElement("link");
      linkElem.rel = kind;
      linkElem.href = url;
      if (as) {
        linkElem.as = as;
      }
      linkElem.crossorigin = true;
      document.head.append(linkElem);
    },
    warmup() {
      if (!this.hasPreconnected) {
        this.addPrefetch("preconnect", "https://www.youtube.com");
        this.addPrefetch("preconnect", "https://www.google.com");
        this.addPrefetch("preconnect", "https://googleads.g.doubleclick.net");
        this.addPrefetch("preconnect", "https://static.doubleclick.net");

        this.hasPreconnected = true;
      }
    },
    click() {
      this.isClicked = true;
    }
  },
  created() {
    this.addPrefetch("preload", this.thumbUrl, "image");
  }
};
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100%;
  height: 0;
  position: relative;
}

iframe,
.thumb {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;

  background: rgba(0, 0, 0, 0.6) no-repeat center center;
  background-size: cover;
  cursor: pointer;
  overflow: hidden;
  display: inline-block;

  &:hover .ytp-large-play-button-bg {
    transition: fill 0.1s cubic-bezier(0, 0, 0.2, 1),
      fill-opacity 0.1s cubic-bezier(0, 0, 0.2, 1);
    fill: #f00;
    fill-opacity: 1;
  }
}

button {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 68px;
  height: 48px;
  transform: translate(-50%, -50%);
  transition: opacity 0.25s cubic-bezier(0, 0, 0.2, 1);
  z-index: 63;
  border: none;
  background-color: transparent;
  padding: 0;
  font-size: 100%;
  cursor: inherit;
  outline: 0;
}
</style>