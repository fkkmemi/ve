<template>
  <v-container fluid>
    <v-card>
      <v-toolbar dense flat color="primary" dark>
        <v-toolbar-title>파일 디비</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn icon @click="read()">
          <v-icon>mdi-read</v-icon>
        </v-btn>
        <v-btn icon @click="fetch()">
          <v-icon>mdi-refresh</v-icon>
        </v-btn>
      </v-toolbar>
      <v-data-table
        :headers="headers"
        :items="items"
      >
        <template v-slot:item.name="{ item }">
          <v-btn @click="execute(item)">{{item.name}}</v-btn>
        </template>
        <template v-slot:item.time="{ item }">
          {{ new Date(item.time).toLocaleString() }}

        </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>
<script>
const { shell } = require('electron')
const { dialog } = require('electron').remote
const fs = require('fs')
const path = require('path')
const Datastore = require('nedb-promises')
const DBFile = Datastore.create('dbFiles.db')

export default {
  data () {
    return {
      dir: '',
      headers: [
        { value: '_id', text: '_id' },
        { value: 'name', text: 'name' },
        // { value: 'path', text: 'path' },
        { value: 'time', text: 'time' },
        { value: 'size', text: 'size' },
        { value: 'tags', text: 'tags' }
      ],
      items: []
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    async fetch () {
      this.items = await DBFile.find()
    },
    async read () {
      const rs = dialog.showOpenDialogSync({
        properties: ['openDirectory']
      })
      if (!rs) return
      this.dir = rs[0]
      const files = fs.readdirSync(this.dir)
      if (!files.length) return
      for (let v of files) {
        const p = path.join(this.dir, v)
        const stat = fs.statSync(p)
        const item = {
          name: v,
          path: p,
          time: stat.ctime,
          size: stat.size,
          tags: []
        }
        const fr = await DBFile.findOne({ path: p })
        if (fr) continue
        await DBFile.insert(item)
      }
      this.fetch()
    },
    execute (item) {
      shell.openItem(item.path)
    }
  }
}
</script>
