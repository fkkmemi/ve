<template>
  <v-container fluid>
    <v-card>
      <v-toolbar dense flat color="primary" dark>
        <v-toolbar-title>문서편집기</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn icon @click="editMode = !editMode">
          <v-icon>{{editMode ? 'mdi-eye' : 'mdi-pencil'}}</v-icon>
        </v-btn>
      </v-toolbar>
      <v-card-text>
        <editor v-if="editMode" v-model="text"></editor>
        <viewer v-else :value="text"></viewer>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn @click="read">read</v-btn>
        <v-btn @click="write">write</v-btn>
      </v-card-actions>

    </v-card>
  </v-container>
</template>
<script>
import 'tui-editor/dist/tui-editor.css'
import 'tui-editor/dist/tui-editor-contents.css'
import 'codemirror/lib/codemirror.css'
import { Editor, Viewer } from '@toast-ui/vue-editor'
const { dialog } = require('electron').remote
const fs = require('fs')

export default {
  components: {
    'editor': Editor,
    'viewer': Viewer
  },
  data () {
    return {
      editMode: true,
      text: ''
    }
  },
  methods: {
    read () {
      const options = {
        filters: [
          {
            name: 'text and markdown',
            extensions: ['txt', 'md']
          }
        ]
      }
      const r = dialog.showOpenDialogSync(options)
      if (!r) return
      this.text = fs.readFileSync(r[0]).toString()
    },
    write () {
      const options = {
        filters: [
          {
            name: 'text and markdown',
            extensions: ['txt', 'md']
          }
        ]
      }
      const r = dialog.showSaveDialogSync(options)
      if (!r) return
      fs.writeFileSync(r, this.text)
    }
  }
}
</script>
