<template>
  <q-page class="flex flex-center">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-vertical.svg"
      style="width: 200px; height: 200px"
    >
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { getRxStorageLoki } from 'rxdb/plugins/lokijs'
import { createRxDatabase } from 'rxdb'
const LokiIncrementalIndexedDBAdapter = require('lokijs/src/incremental-indexeddb-adapter')

export default defineComponent({
  name: 'PageIndex',
  methods: {
    async test () {
      this.rxStorage = getRxStorageLoki({
        adapter: new LokiIncrementalIndexedDBAdapter()
        // * Do not set lokiJS persistence options like autoload and autosave,
        // * RxDB will pick proper defaults based on the given adapter
      })
      this.db = await createRxDatabase({
        name: 'kalpadb',
        storage: this.rxStorage,
        multiInstance: true // <- multiInstance (optional, default: true)
        // eventReduce: false // если поставить true - будут теряться события об обновлении (по всей видимости - это баг)<- eventReduce (optional, default: true)
        // pouchSettings: { revs_limit: 1 }
      }) // 400 msec
    }
  }
})
</script>
