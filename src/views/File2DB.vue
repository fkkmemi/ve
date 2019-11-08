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
          <v-btn @click="execute(item)">{{item.name.substr(0, 20)}}</v-btn>
        </template>
        <template v-slot:item.time="{ item }">
          {{ new Date(item.time).toLocaleString() }}

        </template>
        <template v-slot:item.size="{ item }">
          {{ byte(item.size) }}
        </template>
        <template v-slot:item.tags="{ item }">
          <v-chip v-for="tag in item.tags" :key="tag" close @click:close="delTag(item, tag)">{{tag}}</v-chip>
          <v-btn icon @click="openDialog(item)"><v-icon>mdi-plus</v-icon></v-btn>
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="dialog" width="300">
      <v-card v-if="selectedItem">
        <v-card-title>tag modify</v-card-title>
        <v-card-text>
          <v-combobox
            v-model="selectedItem.tags"
            :items="tags"
            multiple
            chips
            @change="changeTag"
          >

          </v-combobox>
        </v-card-text>
      </v-card>

    </v-dialog>
  </v-container>
</template>
<script>
const { shell } = require('electron')
const { dialog } = require('electron').remote
const fs = require('fs')
const path = require('path')
const byte = require('bytes')
const Datastore = require('nedb-promises')
const DBFile = Datastore.create('dbFiles.db')

export default {
  data () {
    return {
      dir: '',
      byte,
      headers: [
        // { value: '_id', text: '_id' },
        { value: 'name', text: 'name' },
        // { value: 'path', text: 'path' },
        { value: 'time', text: 'time' },
        { value: 'size', text: 'size' },
        { value: 'tags', text: 'tags' }
      ],
      items: [],
      dialog: false,
      selectedItem: null,
      tags: []
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    async fetch () {
      this.items = await DBFile.find()
      this.addTag()
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
    },
    openDialog (item) {
      this.selectedItem = item
      this.dialog = true
    },
    async changeTag () {
      await DBFile.update({ _id: this.selectedItem._id }, { $set: { tags: this.selectedItem.tags } })
      this.addTag()
    },
    addTag () {
      this.items.forEach(item => {
        const tags = item.tags
        tags.forEach(tag => {
          if (!this.tags.includes(tag)) this.tags.push(tag)
        })
      })
    },
    async delTag (item, tag) {
      await DBFile.update({ _id: item._id }, { $pull: { tags: tag } })
      await this.fetch()
    }
  }
}
</script>
