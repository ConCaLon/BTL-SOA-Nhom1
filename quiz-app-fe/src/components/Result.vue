<script setup>
import { NButton } from 'naive-ui'
const { result } = defineProps(['result'])

const goHome = () => {
    window.location.reload()
}
</script>

<template>
    <div class="result-container">
        <div class="result-card">
            <div class="result-icon">🎉</div>
            <h2 class="result-title">Kết Quả Bài Thi</h2>

            <div class="score-circle">
                <span class="score-number">{{ result.score }}</span>
                <span class="score-label">/10</span>
            </div>

            <div class="result-details">
                <div class="detail-row">
                    <span class="detail-label">✅ Số câu đúng:</span>
                    <span class="detail-value correct">{{ result.correct_count }}</span>
                </div>
                <div class="detail-row">
                    <span class="detail-label">❌ Số câu sai:</span>
                    <span class="detail-value wrong">{{ result.total_questions - result.correct_count }}</span>
                </div>
                <div class="detail-row">
                    <span class="detail-label">📝 Tổng số câu:</span>
                    <span class="detail-value">{{ result.total_questions }}</span>
                </div>
            </div>

            <div class="result-message" v-if="result.score >= 5">
                <p style="color: #27ae60;">🎊 Chúc mừng bạn đã đạt!</p>
            </div>
            <div class="result-message" v-else>
                <p style="color: #e74c3c;">😔 Bạn chưa đạt, hãy cố gắng hơn!</p>
            </div>

            <div style="margin-top: 30px;">
                <n-button type="primary" size="large" @click="goHome" style="border-radius: 8px; width: 100%;">
                    Quay lại trang chủ
                </n-button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.result-container {
    width: 100vw;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f0f2f5;
}

.result-card {
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
    text-align: center;
    min-width: 400px;
    max-width: 500px;
}

.result-icon {
    font-size: 48px;
    margin-bottom: 10px;
}

.result-title {
    color: #333;
    margin-bottom: 20px;
    font-size: 24px;
}

.score-circle {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 30px;
    box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4);
}

.score-number {
    color: white;
    font-size: 36px;
    font-weight: bold;
}

.score-label {
    color: rgba(255, 255, 255, 0.8);
    font-size: 18px;
    margin-top: 8px;
}

.result-details {
    text-align: left;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 12px;
    margin-bottom: 20px;
}

.detail-row {
    display: flex;
    justify-content: space-between;
    padding: 8px 0;
    border-bottom: 1px solid #eee;
}

.detail-row:last-child {
    border-bottom: none;
}

.detail-label {
    color: #555;
    font-size: 16px;
}

.detail-value {
    font-weight: bold;
    font-size: 16px;
    color: #333;
}

.detail-value.correct {
    color: #27ae60;
}

.detail-value.wrong {
    color: #e74c3c;
}

.result-message {
    margin-top: 15px;
    font-size: 18px;
    font-weight: bold;
}
</style>
