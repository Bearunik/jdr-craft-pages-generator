<script setup>
import { Coin, SuccessFilled } from '@element-plus/icons-vue'

defineProps({
  title: {
    type: String
  },
  price: {
    type: String
  },
  craftTime: {
    type: String
  },
  rarity: {
    type: String
  },
  difficulty: {
    type: String
  },
  effect: {
    type: String
  },
  components: {
    type: String
  }
})
</script>

<template>
  <article>
    <h3 v-if="title">
      {{ title }}
    </h3>
    <div v-if="rarity">
      <b>Rareté</b>: {{ rarity }}
      <span v-if="price">
        (<el-icon color="orange" :size="12"><Coin /></el-icon> {{ price }})
      </span>
    </div>
    <div v-if="craftTime">
      <b>Temps de création</b>: {{ craftTime }}
      <span v-if="difficulty">
        (<el-icon color="green" :size="12"><SuccessFilled /></el-icon> {{ difficulty }})
      </span>
    </div>
    <div v-if="components"><b>Ingrédients</b>: {{ components }}</div>
    <p v-if="effect">
      {{ effect }}
    </p>
  </article>
</template>

<style scoped>
article {
  margin-bottom: 20px;
}

h3 {
  color: brown;
  margin-bottom: 12px;
  margin-top: 25px;
}
</style>
