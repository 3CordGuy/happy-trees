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
                :options="{ 
                  mode: 'markdown', 
                  lineNumbers: true,
                  autofocus: true
                }"
              ></codemirror>
            </div>
            <div class="column">
              <pre class="output has-text-left" v-html="tree"></pre>
            </div>
            <div class="column">
              <pre class="output has-text-left" v-html="parsed_markdown"></pre>
            </div>
          </div>
          <div class="columns is-centered">
            <button
              class="button is-primary is-rounded is-large copy-button"
              v-text="copy_text"
              @click="on_copy"
            >Copy</button>
          </div>
        </div>
      </div>
    </div>

    <div class="hero-foot is-warning is-bold">
      <footer class="footer">
        <div class="container">
          <div class="content has-text-centered">
            <div>Happy Trees &copy; 2019 Josh Weaver</div>
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
import VueClipboard from "vue-clipboard2";

Vue.use(VueClipboard);

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
      copy_text: "Copy Tree to Clipboard",
      md:
        "# Example\n- assets\n  - imgs\n  - css\n- components\n    - ui-kit\n      - index.js"
    };
  },
  computed: {
    parsed_markdown: function() {
      // return marked.lexer(this.md, { sanitize: true });

      return [
        {
          text: "Example"
        },
        {
          text: "assets",
          list: [
            {
              text: "imgs"
            },
            {
              text: "css"
            }
          ]
        },
        {
          text: "components"
          // list: [
          //   {
          //     text: 'ui-kit',
          //   },
          //   {
          //     text: 'index.js',
          //   }
          // ]
        }
      ];
    },
    tree: function() {
      let tree_nodes = this.parsed_markdown;
      let tree_str = "";
      let depth = 0;
      let parent;
      // Example
      // ├─ assets
      // |  ├─ imgs
      // |  └─ css
      // ├─ components
      // |  ├─ ui-kit
      // |  |  └─ index.js
      function has_next_sibling(arr = [], idx = 0) {
        return arr[idx + 1] ? true : false;
      }

      function get_branch_type(arr, idx) {
        if (has_next_sibling(arr, idx)) return `├─`;
        return `└─`;
      }

      function get_branch_depth(lvl, has_next_sibling = false, parent_has_next_sibling = false) {
        let level = has_next_sibling ? `|  ` : `   `;
        return lvl ? level.repeat(lvl) : ``;
      }

      function climb_tree(nodes, parent = {}) {
        nodes.forEach((node, idx, arr) => {
          // Check if child has a list
          if (node.list) {
            // Start manipulating the string
            tree_str += `${get_branch_depth(
              depth,
              has_next_sibling(nodes, idx),
              parent
            )}${get_branch_type(arr, idx)} ${node.text}\n`;

            depth++;

            // Passing in parent node state as object for downstream checks
            // climb_tree(node.list, { nodes, idx });
          } else if (depth > 0) {
            tree_str += `${
              get_branch_depth(
                depth,
                has_next_sibling(nodes, idx),
                has_next_sibling(parent.nodes, parent.idx)
              )
            }${get_branch_type(arr, idx)} ${node.text}\n`;

          } else {
            tree_str += `${node.text}\n`;
          }
        });
      }

      climb_tree(tree_nodes);

      return tree_str;
    }
  },
  methods: {
    on_copy: function(e) {
      this.$copyText(this.tree).then(
        e => {
          this.copy_text = "Tree Copied! 👍";
          setTimeout(() => (this.copy_text = "Copy Tree to Clipboard"), 3000);
        },
        function(e) {
          alert(`Can't copy! 😱`);
        }
      );
    },
    on_error: function(e) {
      alert("Failed to copy texts");
    },
    update(value) {
      this.md = value;
    }
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
  background: #43c6ac; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to bottom,
    #191654,
    #43c6ac
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to bottom,
    #191654,
    #43c6ac
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
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

.CodeMirror {
  height: 350px;
  background: linear-gradient(
    to left,
    rgba(0, 0, 0, 0.03) 0%,
    rgba(255, 255, 255, 1) 100%
  );
}

pre.output {
  height: 350px;
  border-left: 0.2rem solid #43c6ac;
  font-family: Menlo, Monaco, "Courier New", Courier, monospace;
  line-height: 1.2rem;
  white-space: pre-wrap;
  background: linear-gradient(
    to right,
    rgba(0, 0, 0, 0.03) 0%,
    rgba(255, 255, 255, 1) 100%
  );
}

.main-wrapper {
  background: #ffffff;
  border-radius: 0.8em;
  box-shadow: 0em 0.2em 1em rgba(0, 0, 0, 0.2);
}

.copy-button {
  margin-top: 1.6rem;
}
</style>
