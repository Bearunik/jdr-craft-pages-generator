<script setup>
import { ArrowLeft, UploadFilled, Check } from '@element-plus/icons-vue'
import { ref } from 'vue'
import CraftItem from '../components/CraftItem.vue'

const encodage = 'ISO-8859-1'
const separator = ';'

const ITEM = {
  TITLE: 1,
  RARITY: 2,
  PRICE: 3,
  CRAFTTIME: 4,
  DIFFICULTY: 5,
  COMPONENTS: 6,
  EFFECT: 7
}
const bindedColumnsIndex = [
  { text: 'Sélectionner une valeur', value: 0 },
  { text: 'Titre', value: ITEM.TITLE },
  { text: 'Rareté', value: ITEM.RARITY },
  { text: 'Prix', value: ITEM.PRICE },
  { text: 'Temps de craft', value: ITEM.CRAFTTIME },
  { text: 'DD', value: ITEM.DIFFICULTY },
  { text: 'Effet', value: ITEM.EFFECT },
  { text: 'Ingrédients', value: ITEM.COMPONENTS }
]

let csvFile
let csvFileReaded
let csvArray = []
const csvArrayBindings = ref([])
const bindingDone = ref(false)
const active = ref(0)

function resetAll() {
  active.value = 0
  csvFile = undefined
  csvFileReaded = undefined
  csvArray = []
  csvArrayBindings.value = []
  bindingDone.value = false
}

async function fileSelected(file) {
  active.value = 1
  csvFile = file.raw
  csvFileReaded = await readCSV(csvFile)
  csvArray = arrayGenerator(csvFileReaded, encodage)
  promptMatch(csvArray.header)
}

function promptMatch(csvHeader) {
  const items = csvHeader.map((item) => {
    return { text: item, bindedIndex: 0 }
  })
  csvArrayBindings.value = items
}

function changeBindingStatus(status) {
  bindingDone.value = status
  if (bindingDone.value) {
    active.value = 3
  } else {
    active.value = 1
  }
}

async function readCSV(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = async (e) => {
      try {
        resolve(e.target.result)
      } catch (err) {
        reject(err)
      }
    }
    reader.onerror = (error) => {
      reject(error)
    }
    reader.readAsText(file, encodage)
  })
}

function arrayGenerator(data, encodage) {
  if (encodage === 'ISO-8859-1') {
    const rows = data.split('\r\n')
    return formatHeaderAndBody(rows)
  } else {
    console.log('To implement')
  }
}

function formatHeaderAndBody(rows) {
  let header
  const rowsToRemove = []

  for (let index = 0; index < rows.length; index++) {
    const items = rows[index].split(separator)
    rows[index] = items

    let isDataAvailable = false
    items.forEach((item) => {
      if (item.length > 0) {
        isDataAvailable = true
      }
    })
    if (!isDataAvailable) {
      rowsToRemove.push(index)
    } else if (!header) {
      rowsToRemove.push(index)
      header = rows[index]
    }
  }
  rowsToRemove.sort((a, b) => {
    return b - a
  })
  for (const rowIndex in rowsToRemove) {
    rows.splice(rowsToRemove[rowIndex], 1)
  }
  return { header, body: rows }
}

function getItemBy(row, index) {
  return row[csvArrayBindings.value.findIndex((item) => item.bindedIndex === index)]
}

function print() {
  window.print()
}
</script>

<template>
  <main>
    <h1>Étapes</h1>

    <el-steps style="max-width: 600px" :active="active" finish-status="success" simple>
      <el-step title="Fichier" />
      <el-step title="Binding" />
      <el-step title="Résultat" />
    </el-steps>

    <div v-if="active === 0">
      <h2>Selection du fichier CSV</h2>
      <el-upload
        class="upload-demo"
        drag
        :on-change="fileSelected"
        accept=".csv"
        :auto-upload="false"
      >
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">Déposer votre fichier ici ou <em>cliquer ici</em>.</div>
        <template #tip>
          <div class="el-upload__tip">Seul les fichiers .csv sont acceptés.</div>
        </template>
      </el-upload>
    </div>

    <section v-if="active === 1">
      <h2>Correspondance des colonnes</h2>
      <p>
        <el-alert
          type="info"
          title="Merci de bien vouloir choisir a quel champ correspond chacune de vos colonnes"
          description="Si aucune ne correspond c'est qu'elle n'est pas prise en charge. Dans ce cas, merci de laisser la valeur par défaut."
          show-icon
          :closable="false"
        />
      </p>
      <p>
        <el-alert
          type="warning"
          title="Lien entre certains champs"
          description="Le champ 'prix' est lié à la 'rareté'. Le champ 'difficulté' est au 'temps de craft'. Si non présents ils ne seront pas visibles."
          show-icon
          :closable="false"
        />
      </p>
      <div class="actions">
        <el-button @click="resetAll()" :icon="ArrowLeft" type="danger" round>
          Changer de fichier
        </el-button>
      </div>

      <div v-for="(binding, index) in csvArrayBindings" :key="index" class="promptRow">
        <el-tag type="info">{{ binding.text }}</el-tag>
        <span> >> </span>
        <el-select v-model="binding.bindedIndex" size="small" style="width: 240px">
          <el-option
            v-for="option in bindedColumnsIndex"
            :key="option.value"
            :label="option.text"
            :value="option.value"
          />
        </el-select>
        <span v-if="binding.bindedIndex !== 0">
          <el-icon color="green"><Check /></el-icon>
        </span>
      </div>

      <div class="actions">
        <el-button @click="changeBindingStatus(true)" type="success" round>Valider</el-button>
      </div>
    </section>
    <section v-if="active === 3" id="csvItems">
      <h2>Éléments du CSV</h2>

      <div class="actions">
        <el-button @click="resetAll()" :icon="ArrowLeft" type="danger" round>
          Changer de fichier
        </el-button>
        <el-button @click="changeBindingStatus(false)" :icon="ArrowLeft" round>
          Revoir le binding
        </el-button>
        <el-button @click="print()" type="success" round> Imprimer </el-button>
      </div>

      <CraftItem
        v-for="(row, rowIndex) in csvArray.body"
        :key="rowIndex"
        :title="getItemBy(row, ITEM.TITLE)"
        :price="getItemBy(row, ITEM.PRICE)"
        :craftTime="getItemBy(row, ITEM.CRAFTTIME)"
        :rarity="getItemBy(row, ITEM.RARITY)"
        :difficulty="getItemBy(row, ITEM.DIFFICULTY)"
        :effect="getItemBy(row, ITEM.EFFECT)"
        :components="getItemBy(row, ITEM.COMPONENTS)"
      >
      </CraftItem>
    </section>
  </main>
</template>

<style scoped>
.promptRow {
  padding: 5px 0;
}

.promptRow span {
  margin-left: 5px;
}

.promptColumn {
  padding: 3px;
  background-color: var(--color-background-mute);

  border-radius: 3px;
}

.actions {
  margin: 10px 0;
}

@media print {
  #csvItems {
    background-color: white;
    color: black;
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
    margin: 0;
    padding: 15px;
    font-size: 14px;
    line-height: 18px;
  }

  #csvItems h2,
  button {
    display: none;
  }

  .el-steps {
    display: none;
  }
}
</style>
