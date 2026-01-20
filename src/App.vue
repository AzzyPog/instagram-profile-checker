<script setup>
import {ref} from 'vue'

const followersFile = ref(null)
const followingFile = ref(null)
const result = ref([])
const analyzed = ref(false)
const followerInput = ref(null)
const followingInput = ref(null)


async function readJson (file) {
  return JSON.parse(await file.text())
}

function getNotFollowingBack(followersJson, followingJson) {
  const followers = new Set(
      followersJson.flatMap(item => item.string_list_data.map(d => d.value))
  )

  return followingJson.relationships_following
      .filter(item => !followers.has(item.title))
      .map(item => ({
        username: item.title,
        href: item.string_list_data[0]?.href
      }))
}

async function processFiles() {
  if (!followersFile.value || !followingFile.value) {
    alert("Por favor, selecione ambos os arquivos.")
    return
  }

  try {
    const [followers, following] = await Promise.all([
      readJson(followersFile.value),
      readJson(followingFile.value)
    ])
    result.value = getNotFollowingBack(followers, following)
    analyzed.value = true
  } catch (error) {
    console.error(error)
    alert("Erro ao ler os arquivos. Verifique se são JSONs válidos.")
  }
}
</script>

<template>
  <main class="container">

    <div class="title-subtitle-holder">
      <h1>Instagram Follower Checker</h1>
      <p class="subtitle">
        Veja quem você segue que não te segue de volta sem violar políticas ou usar apps de terceiros.
      </p>
    </div>

    <div v-if="!analyzed" class="upload-section">
      <input hidden type="file" accept=".json" @change="e => followersFile = e.target.files[0]" ref="followerInput"/>
      <input hidden type="file" accept=".json" @change="e => followingFile = e.target.files[0]" ref="followingInput"/>

      <div class="jsonFiles-holder">

        <div class="file-group">
          <button @click="followerInput.click()" class="action-btn">
            📁 JSON dos seguidores
          </button>
          <span v-if="followersFile" class="file-status">✅ {{ followersFile.name }}</span>
        </div>

        <div class="file-group">
          <button @click="followingInput.click()" class="action-btn">
            📁 JSON dos seguindo
          </button>
          <span v-if="followingFile" class="file-status">✅ {{ followingFile.name }}</span>
        </div>

      </div>

      <button @click="processFiles" class="analyze-btn">Analisar</button>
    </div>

    <div v-else>
      <h2>Resultado: {{ result.length }} pessoas não te seguem de volta</h2>
      <button @click="analyzed = false">Voltar</button>
    </div>

  </main>
</template>

<style scoped>

.container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2rem;
  padding: 1rem;
  box-sizing: border-box;
}

.title-subtitle-holder {
  text-align: center;
  max-width: 600px;
}

.subtitle {
  margin-top: 0.5rem;
  line-height: 1.5;
  opacity: 0.9;
}

.upload-section {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

.jsonFiles-holder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;

  background-color: rgba(128, 126, 127, 0.47);
  padding: 2rem;
  border: 1px solid rgba(100, 100, 100, 0.66);
  border-radius: 10px;


  width: 90%;
  max-width: 700px;
}

/* Layout para telas maiores */
@media (min-width: 600px) {
  .jsonFiles-holder {
    flex-direction: row;
    justify-content: space-around;
  }
}

.file-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  width: 100%;
}

.action-btn {
  padding: 10px 20px;
  cursor: pointer;
  font-size: 1rem;
  color: #424242;
  border: 2px solid #7d7d7d91;
  border-radius: 8px;
}

.file-status {
  color: lawngreen;
  font-size: 0.85rem;
  word-break: break-all;
}

.analyze-btn {
  padding: 12px 30px;
  font-size: 1.2rem;
  cursor: pointer;
  background-color: white;
  color: #c405ca;
  border: 3px solid #f8abff;
  border-radius: 5px;
  font-weight: bold;
  transition: transform 0.2s;
}

.analyze-btn:hover {
  transform: scale(1.05);
}
</style>