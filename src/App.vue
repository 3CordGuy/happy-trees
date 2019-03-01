<template>
  <section id="app" class="hero is-fullheight">
    <div class="hero-body">
      <div class="container">
        <div class="section main-wrapper">
          <div class="columns is-centered">
            <div class="column is-half has-text-centered">
              <h1 class="title">🌲 Happy Trees 🌲</h1>
              <h2 class="hero-subtitle has-text-dark">Generate ASCII File Trees from Markdown</h2>
            </div>
          </div>
          <div class="columns is-gapless">
            <div class="column">
              <codemirror
                class="editor"
                v-model="md"
                @change="update"
                :options="{ mode: 'markdown' }"
              ></codemirror>
            </div>
            <div class="column">
              <pre class="output has-text-left" v-html="tree"></pre>
            </div>
          </div>
          <div class="columns is-centered">
            <button class="button is-primary is-rounded is-large copy-button" v-text="copy_text" @click="on_copy">Copy</button>
          </div>
        </div>
      </div>
    </div>

    <div class="hero-foot is-warning is-bold">
      <footer class="footer">
        <div class="container">
          <div class="content has-text-centered">
          <div>
            Happy Trees &copy; 2019 Josh Weaver
          </div>
          <a href="https://github.com/3Cordguy/listen-up">
            <span class="icon is-large">
              <i class="fab fa-2x fa-github-square"></i>
            </span>
          </a>
          <a href="https://www.twitter.com/3CordGuy">
            <span class="icon is-large">
              <i class="fab fa-2x fa-twitter"></i>
            </span>
          </a>
        </div>
        </div>
      </footer>
    </div>
  </section>
</template>

<script>
import marked from "marked";
import _ from "lodash";
import Vue from "vue";
import { codemirror } from "vue-codemirror-lite";
import VueClipboard from 'vue-clipboard2'

Vue.use(VueClipboard)

require("codemirror/mode/markdown/markdown");

const list_mapper = (item, lvl = 1, prev = {}, next = {}) => {
  const level = "|  ";

  const items = {
    heading: `${item.text}\n`,
    list_start: ``,
    list_item_start: `${level.repeat(lvl)}`,
    text: `${next.type === "list_item_start" ? "├─" : "└─"} ${item.text}\n`,
    list_item_end: ``,
    list_end: ``
  };

  if (items[item.type]) return items[item.type];

  if (!items[item.type] && item.text) return item.text;

  return "";
};



export default {
  components: {
    codemirror,
    VueClipboard
  },
  name: "app",
  data() {
    return {
    copy_text: 'Copy Tree to Clipboard',
    md:
        "# Example\n- assets\n  - imgs\n  - css\n- components\n    - ui-kit\n      - index.js"
    };
  },
  computed: {
    parsed_markdown: function() {
      return marked.lexer(this.md, { sanitize: true });
    },
    tree: function() {
      let treeNodes = this.parsed_markdown;
      let str = "";
      let lvl = 0;

      treeNodes.forEach((item, i) => {
        let prevType = treeNodes[i - 2];
        let nextType = treeNodes[i + 2];
        str += list_mapper(item, lvl, prevType, nextType);

        if (item.type === "list_item_start") {
          lvl += 1;
        } else if (item.type === "list_item_end") {
          lvl -= 1;
        }
      });
      return str;
    }
  },
  methods: {
    on_copy: function (e) {
       // : `Couldn't Copy 😱`
      this.$copyText(this.tree)
        .then((e) => {
          this.copy_text = 'Tree Copied! 👍'
          setTimeout(() => (this.copy_text = 'Copy Tree to Clipboard'), 3000)
        }, function (e) {
          alert('Can not copy')
          console.log(e)
        })
    },
    on_error: function (e) {
      alert('Failed to copy texts')
    },
    update(value) {
      this.md = value;
    },
  }
};
</script>

<style lang="scss">
@import "../node_modules/bulma/bulma.sass";
@import url("https://fonts.googleapis.com/css?family=Pacifico");

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  background: #43C6AC;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom, #191654, #43C6AC);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom, #191654, #43C6AC); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */


}

h1.title,
h2.title,
h3.title,
h4.title {
  font-family: "Pacifico", cursive;
}
h1.title {
  font-size: 4em;
}
h2.hero-subtitle {
  font-size: 1.25em;
}


pre.output {
  height: 350px;
  border-left: 2px solid #43C6AC;
  font-family: Menlo, Monaco, "Courier New", Courier, monospace;
  line-height: 20px;
  white-space: pre-wrap;
  background: linear-gradient(to right, rgba(0,0,0,.03) 0%, rgba(255,255,255,1) 100%);
}

.main-wrapper {
    background: #FFFFFF;
    border-radius: 0.8em;
    box-shadow: 0em 0.2em 1em rgba(0, 0, 0, .2);
}

.copy-button {
  margin-top: 1.6rem;
}
</style>
