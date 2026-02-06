<template>
  <div class="app">
    <div class="container">
      <h1 class="title">🎯 공부 타이머</h1>
      
      <!-- 타이머 디스플레이 -->
      <div class="timer-display">
        <div class="phase-label">{{ phaseLabel }}</div>
        <div class="time">{{ formattedTime }}</div>
        <div class="set-info">{{ currentSet }} / 10 세트</div>
      </div>

      <!-- 시작/일시정지 버튼 -->
      <div class="controls">
        <button 
          v-if="!isStarted" 
          @click="startTimer" 
          class="btn btn-start"
        >
          START
        </button>
        <template v-else>
          <button 
            v-if="!isPaused" 
            @click="pauseTimer" 
            class="btn btn-pause"
          >
            일시정지
          </button>
          <button 
            v-else 
            @click="resumeTimer" 
            class="btn btn-resume"
          >
            재개
          </button>
          <button 
            @click="resetTimer" 
            class="btn btn-reset"
          >
            리셋
          </button>
        </template>
      </div>

      <!-- 세트 진행 게이지 -->
      <div class="progress-container">
        <div class="progress-label">세트 진행도</div>
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :style="{ width: progressPercentage + '%' }"
          ></div>
        </div>
        <div class="progress-sets">
          <div 
            v-for="n in 10" 
            :key="n" 
            class="set-indicator"
            :class="{ 
              'completed': n < currentSet, 
              'current': n === currentSet,
              'upcoming': n > currentSet
            }"
          >
            {{ n }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

// 타이머 설정 (실제 사용)
const PREPARE_TIME = 5     // 준비 시간 (5초)
const STUDY_TIME = 3000    // 공부 시간 (50분 = 3000초)
const BREAK_TIME = 600     // 휴식 시간 (10분 = 600초)
const TOTAL_SETS = 10      // 총 세트 수

// 상태 관리
const isStarted = ref(false)
const isPaused = ref(false)
const currentPhase = ref('prepare') // 'prepare', 'study', 'break'
const currentSet = ref(1)
const timeLeft = ref(PREPARE_TIME)
let timerInterval = null

// 페이즈 라벨
const phaseLabel = computed(() => {
  switch (currentPhase.value) {
    case 'prepare':
      return '⏰ 준비'
    case 'study':
      return '📚 공부 시간'
    case 'break':
      return '☕ 휴식 시간'
    default:
      return ''
  }
})

// 시간 포맷팅
const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
  const seconds = timeLeft.value % 60
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

// 진행률 계산
const progressPercentage = computed(() => {
  return ((currentSet.value - 1) / TOTAL_SETS) * 100
})

// 타이머 시작
const startTimer = () => {
  isStarted.value = true
  isPaused.value = false
  currentPhase.value = 'prepare'
  timeLeft.value = PREPARE_TIME
  runTimer()
}

// 타이머 실행
const runTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
  
  timerInterval = setInterval(() => {
    if (!isPaused.value) {
      timeLeft.value--
      
      if (timeLeft.value <= 0) {
        moveToNextPhase()
      }
    }
  }, 1000)
}

// 다음 페이즈로 이동
const moveToNextPhase = () => {
  if (currentPhase.value === 'prepare') {
    // 준비 → 공부
    currentPhase.value = 'study'
    timeLeft.value = STUDY_TIME
    playSound()
  } else if (currentPhase.value === 'study') {
    // 공부 → 휴식
    currentPhase.value = 'break'
    timeLeft.value = BREAK_TIME
    playSound()
  } else if (currentPhase.value === 'break') {
    // 휴식 → 다음 세트 또는 종료
    if (currentSet.value < TOTAL_SETS) {
      currentSet.value++
      currentPhase.value = 'prepare'
      timeLeft.value = PREPARE_TIME
      playSound()
    } else {
      // 10세트 완료
      completeAllSets()
    }
  }
}

// 모든 세트 완료
const completeAllSets = () => {
  clearInterval(timerInterval)
  alert('🎉 축하합니다! 10세트를 모두 완료했습니다!')
  resetTimer()
}

// 알림음 재생 (브라우저 기본 알림)
const playSound = () => {
  // 간단한 알림음 (실제로는 오디오 파일을 사용할 수 있습니다)
  if (typeof window !== 'undefined' && window.AudioContext) {
    const audioContext = new AudioContext()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()
    
    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)
    
    oscillator.frequency.value = 800
    gainNode.gain.value = 0.3
    
    oscillator.start()
    setTimeout(() => oscillator.stop(), 200)
  }
}

// 일시정지
const pauseTimer = () => {
  isPaused.value = true
}

// 재개
const resumeTimer = () => {
  isPaused.value = false
}

// 리셋
const resetTimer = () => {
  clearInterval(timerInterval)
  isStarted.value = false
  isPaused.value = false
  currentPhase.value = 'prepare'
  currentSet.value = 1
  timeLeft.value = PREPARE_TIME
}

// 컴포넌트 언마운트 시 타이머 정리
onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})
</script>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.container {
  background: white;
  border-radius: 30px;
  padding: 40px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 600px;
  width: 100%;
}

.title {
  text-align: center;
  color: #667eea;
  font-size: 2.5rem;
  margin-bottom: 30px;
  font-weight: bold;
}

.timer-display {
  text-align: center;
  margin-bottom: 40px;
  padding: 30px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 20px;
}

.phase-label {
  font-size: 1.5rem;
  color: #667eea;
  font-weight: 600;
  margin-bottom: 20px;
}

.time {
  font-size: 5rem;
  font-weight: bold;
  color: #2d3748;
  font-family: 'Courier New', monospace;
  margin: 20px 0;
}

.set-info {
  font-size: 1.2rem;
  color: #718096;
  margin-top: 15px;
}

.controls {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 40px;
}

.btn {
  padding: 15px 40px;
  font-size: 1.2rem;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.btn-start {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px 60px;
  font-size: 1.5rem;
}

.btn-pause {
  background: #ed8936;
  color: white;
}

.btn-resume {
  background: #48bb78;
  color: white;
}

.btn-reset {
  background: #f56565;
  color: white;
}

.progress-container {
  margin-top: 30px;
}

.progress-label {
  text-align: center;
  font-size: 1.1rem;
  color: #4a5568;
  margin-bottom: 15px;
  font-weight: 600;
}

.progress-bar {
  width: 100%;
  height: 30px;
  background: #e2e8f0;
  border-radius: 15px;
  overflow: hidden;
  margin-bottom: 20px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transition: width 0.3s ease;
  border-radius: 15px;
}

.progress-sets {
  display: flex;
  justify-content: space-between;
  gap: 5px;
}

.set-indicator {
  flex: 1;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  font-weight: bold;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.set-indicator.completed {
  background: #48bb78;
  color: white;
}

.set-indicator.current {
  background: #667eea;
  color: white;
  transform: scale(1.1);
  box-shadow: 0 4px 10px rgba(102, 126, 234, 0.5);
}

.set-indicator.upcoming {
  background: #e2e8f0;
  color: #a0aec0;
}

@media (max-width: 600px) {
  .title {
    font-size: 2rem;
  }
  
  .time {
    font-size: 3.5rem;
  }
  
  .btn {
    padding: 12px 30px;
    font-size: 1rem;
  }
  
  .btn-start {
    padding: 15px 40px;
    font-size: 1.2rem;
  }
  
  .set-indicator {
    font-size: 0.8rem;
    height: 40px;
  }
}
</style>
