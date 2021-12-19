<template>
  <q-page class="flex flex-center">
    <q-btn
        icon="clear"
        label="CLEAR DB"
        color="red"
        @click="onBtnClick"
    />
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { getRxStorageLoki } from 'rxdb/plugins/lokijs'
import { createRxDatabase, addRxPlugin } from 'rxdb/plugins/core'
import { RxDBAjvValidatePlugin } from 'rxdb/plugins/ajv-validate'
import { RxDBLeaderElectionPlugin } from 'rxdb/plugins/leader-election'
import { RxDBUpdatePlugin } from 'rxdb/plugins/update'

addRxPlugin(RxDBUpdatePlugin)
import { RxDBDevModePlugin } from 'rxdb/plugins/dev-mode'

addRxPlugin(RxDBDevModePlugin)

addRxPlugin(RxDBLeaderElectionPlugin)
addRxPlugin(RxDBAjvValidatePlugin)
const LokiIncrementalIndexedDBAdapter = require('lokijs/src/incremental-indexeddb-adapter')

export default defineComponent({
  name: 'PageIndex',
  async mounted () { await this.createDB() },
  methods: {
    async createDB () { // executes on every page load
      if (!this.db) {
        this.rxStorage = getRxStorageLoki({
          adapter: new LokiIncrementalIndexedDBAdapter()
        })
        this.db = await createRxDatabase({
          name: 'db',
          multiInstance: true,
          // ignoreDuplicate: true,
          storage: this.rxStorage
        })
      }
      await this.db.addCollections({
        // key = collectionName
        humans: {
          schema: {
            type: 'object',
            version: 0,
            primaryKey: 'name',
            properties: {
              name: {
                type: 'string'
              },
              name2: {
                type: 'string'
              }
            }
          }
        }
      })
      console.log('before exec')
      const docs = await this.db.humans.find().exec()
      console.log('after exec', docs.length)
    },
    async onBtnClick () { // click the button on second tab
      await this.db.humans.remove() // remove collection
      await this.createDB() // 'after exec' will never printed on second tab
    }
  }
})
</script>
