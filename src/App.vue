<template lang="pug">
#app.app
  .app__sheet
    .page
      .page__pair(v-for="(token, index) in replicatedTokens" :key="token.id + index")
        .page__item(:class="getTokenClass(token.size)" :style="getTokenStyle(token)")
          .page__badge(v-if="token.count > 1") {{index + 1}}
        .page__item(:class="getTokenClass(token.size)" :style="getTokenStyle(token)")
          .page__badge(v-if="token.count > 1") {{index + 1}}
  .app__controls
    h1 Token Sheet Creator
    ol
      li Drop your images below
      li Adjust size, count and align
      li Print it or save as .pdf
    .drop-area(ref="dropArea", :class="{'drop-area--highlighted': isFileOverArea}")
    .token-control(v-for="(token, id) in tokens")
      .token-control__preview.align
        .align__scene
          .align__area(
            v-for="(align, alignId) in aligns"
            :class="{'align__area--active': token.align === alignId}"
            @click="setAlign(id, alignId)"
            )
            .align__icon
              icon(:glyph="getGlyph(align)" :width="15" :height="15")
        img.token-control__image(:src="token.src")
      .token-control__actions.actions
        .actions__row
          button(@click="setCount(id, 1)").actions__button +
          span.actions__count {{token.count}}
          button(@click="setCount(id, -1)").actions__button -
          button(@click="removeToken(id)").actions__button.actions__button--end &times;
        .actions__row
          .actions__label(
            v-for="(size, sizeId) in sizes"
            @click="setSize(id, sizeId)"
            :class="{'actions__label--active': token.size === sizeId}") {{size}}
        .actions__row
          .actions__label(
            @click="setCover(id, false)"
            :class="{'actions__label--active': token.isCover === false}") Fit
          .actions__label(
            @click="setCover(id, true)"
            :class="{'actions__label--active': token.isCover === true}") Cover
</template>

<script>
import nanoid from 'nanoid';
import Vue from 'vue';
import Icon from '@/components/icon/';
import './styles/main.scss';

export default {
  name: 'app',
  components: { Icon },
  data() {
    return {
      sizes: {
        medium: 'Medium',
        large: 'Large',
        huge: 'Huge',
        gargantuan: 'Gargantuan',
      },
      aligns: {
        topLeft: 'top left',
        topCenter: 'top center',
        topRight: 'top right',
        centerLeft: 'center left',
        centerCenter: 'center center',
        centerRight: 'center right',
        bottomLeft: 'bottom left',
        bottomCenter: 'bottom center',
        bottomRight: 'bottom right',
      },
      tokens: {},
      isFileOverArea: false,
    };
  },
  computed: {
    replicatedTokens() {
      let tokens = [];
      Object.values(this.tokens).forEach((token) => {
        for (let i = 0; i < token.count; i++) {
          tokens = [...tokens, token];
        }
      });
      return tokens;
    },
  },
  mounted() {
    this.$nextTick(this.init);
  },
  methods: {
    setCount(id, amount) {
      const newCount = this.tokens[id].count + amount;
      if (newCount <= 0) {
        this.removeToken(id);
      } else {
        this.tokens[id].count = newCount;
      }
    },
    setAlign(id, alignId) {
      this.tokens[id].align = alignId;
    },
    setSize(id, sizeId) {
      this.tokens[id].size = sizeId;
    },
    setCover(id, value) {
      this.tokens[id].isCover = value;
    },
    createToken(data) {
      const id = nanoid();
      Vue.set(this.tokens, id, {
        id,
        src: data,
        count: 1,
        size: 'medium',
        isCover: true,
        align: 'centerCenter',
      });
    },
    removeToken(id) {
      Vue.delete(this.tokens, id);
    },
    getTokenClass(size) {
      return `page__item--${size}`;
    },
    getTokenStyle(token) {
      return `
        background-size: ${token.isCover ? 'cover' : 'contain'};
        background-position: ${this.aligns[token.align]};
        background-image: url(${token.src});
        `;
    },
    getGlyph(string) {
      return string.replace(' ', '-');
    },
    init() {
      const { dropArea } = this.$refs;

      function preventDefaults(e) {
        e.preventDefault();
        e.stopPropagation();
      }

      const previewFile = (file) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onloadend = () => {
          this.createToken(reader.result);
        };
      };

      function handleFiles(files) {
        [...files].forEach(previewFile);
      }

      function handleDrop(e) {
        const dt = e.dataTransfer;
        const { files } = dt;
        handleFiles(files);
      }


      ['dragenter', 'dragover', 'dragleave', 'drop'].forEach((eventName) => {
        dropArea.addEventListener(eventName, preventDefaults, false);
      });

      ['dragover', 'drop'].forEach((eventName) => {
        window.addEventListener(eventName, preventDefaults, false);
      });

      ['dragenter', 'dragover'].forEach((eventName) => {
        dropArea.addEventListener(eventName, () => {
          this.isFileOverArea = true;
        }, false);
      });

      ['dragleave', 'drop'].forEach((eventName) => {
        dropArea.addEventListener(eventName, () => {
          this.isFileOverArea = false;
        }, false);
      });

      dropArea.addEventListener('drop', handleDrop, false);
    },
  },
};
</script>
