<template>
  <section id="app" class="hero is-light is-bold is-fullheight">
    <div class="hero-body">
      <div class="container">
        <h1 class="title">🌲 Happy Trees 🌲</h1>
        <h2 class="hero-subtitle has-text-dark">Generate ASCII File Trees from Markdown</h2>
        <div class="section">
          <div class="columns">
            <div class="column">
              <textarea class="textarea" v-model="md" @md="update" placeholder=""></textarea>
            </div>
            <div class="column">
              <pre class="content has-text-left" v-html="tree"></pre>
            </div>
            <div class="column">
              <pre class="content has-text-left" v-html="compiledMarkdown"></pre>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Editor from './components/Editor.vue'
import marked from 'marked'
import _ from 'lodash'

const listMapper = (item, lvl = 1, prev = {}, next = {}) => {
  console.log(item, lvl, next)
  const level = '  |  '
  const prevStruct = prev.type === 'list_start' ? level.repeat(lvl) : '    '
  const nextStruct = next.type === 'list_start' ? '└─' : '├─'
  const items = {
    heading: `${item.text}`,
    list_start: `  `,
    list_item_start: `\n${lvl > 0 ? prevStruct : ''}  ${nextStruct}`,
    text: ` ${item.text}`,
    list_item_end: ``,
    list_end: ``
  }

  return items[item.type] ? items[item.type] : ''
}

export default {
  name: 'app',
  data() {
    return {
      md: '# directory\n- file.png\n- tests\n  - file.jpg'
    }
  },
  computed: {
    compiledMarkdown: function () {
      return marked.lexer(this.md, { sanitize: true })
    },
    tree: function () {
      let treeNodes = this.compiledMarkdown
      let str = ''
      let lvl = 0
      treeNodes.forEach((item, i) => {
        let prevType = treeNodes[i-2]
        let nextType = treeNodes[i+2]
        str += listMapper(item, lvl, prevType, nextType)

        if (item.type === 'list_item_start') {
          lvl += 1
        } else if (item.type === 'list_item_end') {
          lvl -= 1
        }
      })
      return str
    }
  },
  methods: {
    update: _.debounce(function (e) {
      this.md = e.target.value
    }, 300)
  }
}

</script>

<style lang="scss">
@import '../node_modules/bulma/bulma.sass';
@import url('https://fonts.googleapis.com/css?family=Pacifico');

#app {
	font-family: 'Avenir', Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
}

h1.title,
h2.title,
h3.title,
h4.title {
	font-family: 'Pacifico', cursive;
}
h1.title {
	font-size: 4em;
}
h2.hero-subtitle {
	font-size: 1.25em;
}
</style>
