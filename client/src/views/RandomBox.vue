<template>
  <div>
    <div style="text-align:center;margin-bottom:32px;">
      <h1 style="color:white;margin-bottom:8px;">🎰 随机拆盒机</h1>
      <p style="color:rgba(255,255,255,0.8);">点击按钮，随机抽取一个神秘盲盒</p>
    </div>

    <div v-if="noItems" class="empty-state card" style="text-align:center;">
      <h2>暂无可拆盲盒</h2>
      <p style="margin-bottom:20px;">目前没有其他用户发布的可交换盲盒</p>
      <router-link to="/publish">
        <button class="btn btn-primary">去发布一个</button>
      </router-link>
    </div>

    <template v-else>
      <div class="flip-container" :class="{ 'flipped': revealed }" @click="handleCardClick">
        <div class="flip-card">
          <div class="flip-front">
            <div style="text-align:center;">
              <div style="font-size:80px;margin-bottom:16px;">🎁</div>
              <p style="color:white;font-size:20px;font-weight:600;">
                {{ isFirstDraw ? '点击开始拆盒' : '点击拆开盲盒' }}
              </p>
            </div>
          </div>
          <div class="flip-back">
            <div v-if="currentItem" style="text-align:center;">
              <div style="width:160px;height:160px;border-radius:12px;overflow:hidden;margin:0 auto 16px;background:#f0f0f0;">
                <img :src="appendAuth(currentItem.image)" alt="盲盒图片"
                     style="width:100%;height:100%;object-fit:cover;"/>
              </div>
              <h3 style="font-size:18px;margin-bottom:8px;">{{ getCategoryName(currentItem.category) }}</h3>
              <div style="margin-bottom:12px;">
                <span v-for="tag in currentItem.mysteryTags" :key="tag" class="tag" style="font-size:13px;padding:5px 12px;">
                  {{ tag }}
                </span>
              </div>
              <p style="color:#999;font-size:12px;">发布于 {{ formatDate(currentItem.createdAt) }}</p>
            </div>
          </div>
        </div>
      </div>

      <div style="text-align:center;margin-top:32px;display:flex;gap:16px;justify-content:center;flex-wrap:wrap;">
        <button class="btn btn-primary" style="padding:14px 36px;font-size:16px;" @click="handleRandom" :disabled="loading">
          {{ loading ? '抽取中...' : (isFirstDraw ? '🎰 开始拆盒' : '🎲 再来一次') }}
        </button>
        <router-link v-if="currentItem && revealed" :to="'/item/' + currentItem.id">
          <button class="btn btn-secondary" style="padding:14px 36px;font-size:16px;">
            🔍 查看详情并交换
          </button>
        </router-link>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getRandomItem, appendAuth } from '../api/index.js'
import { userStore } from '../store/user.js'

const currentItem = ref(null)
const revealed = ref(false)
const loading = ref(false)
const noItems = ref(false)
const isFirstDraw = ref(true)

const categories = {
  book: '书籍类',
  figure: '手办类',
  toy: '玩具类',
  game: '游戏类',
  digital: '数码类',
  other: '其他'
}

function getCategoryName(key) {
  return categories[key] || key
}

function formatDate(dateStr) {
  const date = new Date(dateStr)
  const y = date.getFullYear()
  const m = String(date.getMonth() + 1).padStart(2, '0')
  const d = String(date.getDate()).padStart(2, '0')
  return y + '-' + m + '-' + d
}

function ensureUserId() {
  if (!userStore.user || !userStore.user.id) {
    const newId = 'user_' + Math.random().toString(36).substr(2, 16)
    const newName = '用户' + Math.floor(Math.random() * 10000)
    userStore.user.id = newId
    userStore.user.name = newName
    localStorage.setItem('blind_box_user', JSON.stringify({ id: newId, name: newName }))
  }
  return userStore.user.id
}

function handleCardClick() {
  if (!revealed.value) {
    if (isFirstDraw.value) {
      handleRandom()
    } else if (currentItem.value) {
      revealed.value = true
    }
  }
}

async function handleRandom() {
  const userId = ensureUserId()
  if (!userId) {
    alert('用户信息异常，请刷新页面重试')
    return
  }

  loading.value = true
  revealed.value = false
  try {
    const item = await getRandomItem(userId)
    if (!item) {
      noItems.value = true
      return
    }
    currentItem.value = item
    noItems.value = false
    isFirstDraw.value = false
    setTimeout(function() {
      revealed.value = true
    }, 600)
  } catch (e) {
    console.error(e)
    alert('抽取失败，请稍后重试')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.flip-container {
  perspective: 1200px;
  width: 320px;
  height: 420px;
  margin: 0 auto;
  cursor: pointer;
}

.flip-card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.flip-container.flipped .flip-card {
  transform: rotateY(180deg);
}

.flip-front,
.flip-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.flip-front {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.flip-back {
  background: white;
  transform: rotateY(180deg);
  padding: 24px;
}
</style>
