<template>
  <header class="app-header" :class="{ scrolled: isScrolled }">
    <div class="header-container">
      <div class="logo" @click="router.push('/')">
        <div class="logo-icon">
          <svg width="32" height="32" viewBox="0 0 32 32" fill="none">
            <rect width="32" height="32" rx="8" fill="url(#logo-gradient)"/>
            <path d="M10 16L14 12L18 16L22 12" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M10 20L14 16L18 20L22 16" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" opacity="0.6"/>
            <defs>
              <linearGradient id="logo-gradient" x1="0" y1="0" x2="32" y2="32">
                <stop stop-color="#6366f1"/>
                <stop offset="1" stop-color="#a855f7"/>
              </linearGradient>
            </defs>
          </svg>
        </div>
        <span class="logo-text">Portal</span>
      </div>
      
      <nav class="nav-menu hidden-mobile">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-item"
          :class="{ active: isActive(item.path) }"
        >
          {{ item.name }}
        </router-link>
      </nav>

      <div class="header-actions">
        <div class="search-wrapper">
          <el-button class="search-btn" :class="{ active: showSearch }" circle @click="toggleSearch">
            <el-icon><Search /></el-icon>
          </el-button>
          
          <transition name="fade">
            <div v-if="showSearch" class="search-panel" @click.stop>
              <div class="search-input-wrapper">
                <el-input
                  v-model="searchKeyword"
                  placeholder="输入搜索内容..."
                  :prefix-icon="Search"
                  clearable
                  size="large"
                  class="search-input"
                  @keyup.enter="handleSearch"
                  ref="searchInputRef"
                />
                <el-button type="primary" class="search-submit-btn" @click="handleSearch">
                  搜索
                </el-button>
              </div>
              
              <div v-if="searchKeyword && hotSearches.length > 0" class="search-suggestions">
                <div class="suggestions-title">搜索建议</div>
                <div
                  v-for="item in hotSearches"
                  :key="item.id"
                  class="suggestion-item"
                  @click="selectSuggestion(item.title)"
                >
                  <el-icon class="suggestion-icon"><Search /></el-icon>
                  <span>{{ item.title }}</span>
                </div>
              </div>
            </div>
          </transition>
        </div>
        
        <el-button class="contact-btn hidden-mobile" type="primary" round @click="router.push('/contact')">
          开始咨询
          <el-icon class="btn-arrow"><Right /></el-icon>
        </el-button>
        <div class="mobile-menu-btn hidden-desktop" @click="toggleMobileMenu">
          <span :class="{ open: mobileMenuVisible }"></span>
        </div>
      </div>

      <transition name="fade">
        <div v-if="showSearch" class="search-overlay" @click="closeSearch"></div>
      </transition>
    </div>

    <!-- 移动端菜单 -->
    <transition name="slide-fade">
      <div v-if="mobileMenuVisible" class="mobile-menu hidden-desktop">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="mobile-nav-item"
          @click="mobileMenuVisible = false"
        >
          {{ item.name }}
          <el-icon><Right /></el-icon>
        </router-link>
      </div>
    </transition>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { Search, Right } from '@element-plus/icons-vue'
import type { NavItem } from '@/types'
import { ElMessage } from 'element-plus'

const router = useRouter()
const route = useRoute()

const mobileMenuVisible = ref(false)
const isScrolled = ref(false)
const showSearch = ref(false)
const searchKeyword = ref('')
const searchInputRef = ref<HTMLElement | null>(null)

interface HotSearchItem {
  id: number
  title: string
}

const hotSearches: HotSearchItem[] = [
  { id: 1, title: '产品介绍' },
  { id: 2, title: '新闻动态' },
  { id: 3, title: '联系方式' },
  { id: 4, title: '关于我们' },
]

const navItems: NavItem[] = [
  { name: '首页', path: '/' },
  { name: '关于我们', path: '/about' },
  { name: '新闻动态', path: '/news' },
  { name: '产品服务', path: '/products' },
]

const isActive = (path: string) => {
  if (path === '/') return route.path === '/'
  return route.path.startsWith(path)
}

const toggleMobileMenu = () => {
  mobileMenuVisible.value = !mobileMenuVisible.value
}

const toggleSearch = () => {
  showSearch.value = !showSearch.value
  if (showSearch.value) {
    nextTick(() => {
      searchInputRef.value?.focus()
    })
  }
}

const closeSearch = () => {
  showSearch.value = false
  searchKeyword.value = ''
}

const handleSearch = () => {
  if (!searchKeyword.value.trim()) {
    ElMessage.warning('请输入搜索内容')
    return
  }
  
  ElMessage.success(`正在搜索: ${searchKeyword.value}`)
  
  closeSearch()
}

const selectSuggestion = (keyword: string) => {
  searchKeyword.value = keyword
  handleSearch()
}

const handleScroll = () => {
  isScrolled.value = window.scrollY > 20
}

const handleClickOutside = (e: MouseEvent) => {
  const target = e.target as HTMLElement
  if (showSearch.value && !target.closest('.search-wrapper')) {
    closeSearch()
  }
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  document.removeEventListener('click', handleClickOutside)
})
</script>

<style lang="scss" scoped>
.app-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  transition: all $transition-normal;
  
  &.scrolled {
    background: rgba(255, 255, 255, 0.8);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  }
}

.header-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: $container-max-width;
  margin: 0 auto;
  padding: 0 $spacing-lg;
  height: $header-height;
}

.logo {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  cursor: pointer;
  
  &-text {
    font-size: $font-size-xl;
    font-weight: 700;
    background: $gradient-text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
}

.nav-menu {
  display: flex;
  gap: $spacing-xs;
}

.nav-item {
  position: relative;
  padding: $spacing-sm $spacing-md;
  font-size: $font-size-sm;
  font-weight: 500;
  color: $text-color-regular;
  border-radius: $border-radius-full;
  transition: all $transition-fast;

  &:hover {
    color: $text-color-primary;
    background: $bg-color-light;
  }

  &.active {
    color: $primary-color;
    background: rgba($primary-color, 0.1);
  }
}

.header-actions {
  display: flex;
  align-items: center;
  gap: $spacing-md;
}

.contact-btn {
  background: $gradient-primary;
  border: none;
  font-weight: 600;
  
  .btn-arrow {
    margin-left: 4px;
    transition: transform $transition-fast;
  }
  
  &:hover .btn-arrow {
    transform: translateX(4px);
  }
}

.mobile-menu-btn {
  width: 24px;
  height: 20px;
  position: relative;
  cursor: pointer;
  
  span,
  span::before,
  span::after {
    position: absolute;
    width: 100%;
    height: 2px;
    background: $text-color-primary;
    border-radius: 2px;
    transition: all $transition-fast;
  }
  
  span {
    top: 50%;
    transform: translateY(-50%);
    
    &::before {
      content: '';
      top: -7px;
    }
    
    &::after {
      content: '';
      top: 7px;
    }
    
    &.open {
      background: transparent;
      
      &::before {
        top: 0;
        transform: rotate(45deg);
      }
      
      &::after {
        top: 0;
        transform: rotate(-45deg);
      }
    }
  }
}

.mobile-menu {
  position: absolute;
  top: $header-height;
  left: 0;
  right: 0;
  background: $bg-color-white;
  border-bottom: 1px solid $border-color-light;
  padding: $spacing-md;
  box-shadow: $shadow-lg;
}

.mobile-nav-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: $spacing-md;
  font-size: $font-size-md;
  font-weight: 500;
  color: $text-color-primary;
  border-radius: $border-radius-md;
  
  &:hover {
    background: $bg-color-light;
    color: $primary-color;
  }
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all $transition-normal;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity $transition-fast;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.search-wrapper {
  position: relative;
}

.search-btn {
  background: transparent;
  border: none;
  color: $text-color-regular;
  transition: all $transition-fast;

  &:hover,
  &.active {
    background: rgba($primary-color, 0.1);
    color: $primary-color;
  }
}

.search-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  z-index: 999;
}

.search-panel {
  position: absolute;
  top: calc(100% + 12px);
  right: 0;
  width: 420px;
  background: $bg-color-white;
  border-radius: $border-radius-lg;
  box-shadow: $shadow-xl;
  padding: $spacing-md;
  z-index: 1001;
  transform-origin: top right;
  animation: slideDown 0.2s ease;

  @media (max-width: $breakpoint-md) {
    position: fixed;
    top: $header-height;
    left: $spacing-md;
    right: $spacing-md;
    width: auto;
  }
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.search-input-wrapper {
  display: flex;
  gap: $spacing-sm;
  align-items: center;
}

.search-input {
  flex: 1;

  :deep(.el-input__wrapper) {
    box-shadow: none;
    border: 1px solid $border-color-light;
    transition: all $transition-fast;

    &:hover,
    &.is-focus {
      border-color: $primary-color;
    }
  }
}

.search-submit-btn {
  background: $gradient-primary;
  border: none;
  font-weight: 600;
}

.search-suggestions {
  margin-top: $spacing-md;
  padding-top: $spacing-md;
  border-top: 1px solid $border-color-light;
}

.suggestions-title {
  font-size: $font-size-xs;
  color: $text-color-placeholder;
  margin-bottom: $spacing-sm;
  font-weight: 500;
}

.suggestion-item {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  padding: $spacing-sm $spacing-md;
  border-radius: $border-radius-md;
  cursor: pointer;
  transition: all $transition-fast;
  font-size: $font-size-sm;
  color: $text-color-regular;

  &:hover {
    background: $bg-color-light;
    color: $primary-color;
  }

  .suggestion-icon {
    font-size: 16px;
    color: $text-color-placeholder;
  }
}
</style>
