<template>
    <main class="questions-wrapper">
        <div class="exam-layout">
            <!-- Cột Danh sách câu hỏi -->
            <div class="questions-column">
                <div class="list" v-for="(ques, index) in questions" :key="ques._id || index" :id="`question-${index}`">
                    <div class="question-container">
                        <div class="question">Câu {{ index + 1 }}: {{ ques.text }}</div>
                    </div>

                    <div class="answers-container">
                        <n-radio-group :value="selectedOptions[ques._id]"
                            @update:value="(val) => selectOption(ques._id, val)" name="optionGroup">
                            <n-grid cols="1" x-gap="12" y-gap="12">
                                <n-gi v-for="(option, idx) in ques.answers" :key="idx">
                                    <div class="radio-card" :class="{ 'selected': selectedOptions[ques._id] === option.text }" @click="selectOption(ques._id, option.text)">
                                        <n-radio :value="option.text"
                                            :label="`${String.fromCharCode(65 + idx)}. ${option.text}`" />
                                    </div>
                                </n-gi>
                            </n-grid>
                        </n-radio-group>
                    </div>
                </div>

                <div class="btn-end">
                    <n-button class="submit-btn" size="large" @click="handleSubmit">
                        Nộp Bài
                    </n-button>
                </div>
            </div>

            <!-- Cột Checklist -->
            <div class="checklist-column">
                <div class="checklist-card">
                    <h3 class="checklist-title">Danh sách câu hỏi</h3>
                    <div class="checklist-grid">
                        <div 
                            v-for="(ques, index) in questions" 
                            :key="index"
                            class="checklist-item"
                            :class="{ 'answered': selectedOptions[ques._id] }"
                            @click="scrollToQuestion(index)"
                        >
                            {{ index + 1 }}
                        </div>
                    </div>
                    <div class="checklist-status">
                        <div class="status-item"><div class="box answered-box"></div> Đã làm</div>
                        <div class="status-item"><div class="box default-box"></div> Chưa làm</div>
                    </div>
                </div>
            </div>
        </div>
    </main>
</template>

<script setup>
import { ref, defineProps, defineEmits } from "vue"

const { questions } = defineProps(['questions'])
const emit = defineEmits(['submit', 'update:answers'])

const selectedOptions = ref({})

function selectOption(questionId, optionText)
{
    selectedOptions.value[questionId] = optionText
    // Emit ra mảng đáp án để Main.vue theo dõi
    const answersArray = Object.entries(selectedOptions.value).map(([qId, ans]) => ({
        question_id: qId,
        selected_answer: ans
    }))
    emit('update:answers', answersArray)
}

function scrollToQuestion(index) {
    const el = document.getElementById(`question-${index}`)
    if (el) {
        // Trừ đi chiều cao của header (khoảng 80px) để không bị che
        const y = el.getBoundingClientRect().top + window.scrollY - 100
        window.scrollTo({ top: y, behavior: 'smooth' })
    }
}

function handleSubmit()
{
    emit('submit')
}
</script>

<style scoped>
.questions-wrapper {
    padding-bottom: 50px;
}

.exam-layout {
    display: flex;
    gap: 24px;
    align-items: flex-start;
}

.questions-column {
    flex: 3;
}

.checklist-column {
    flex: 1;
    position: sticky;
    top: 100px; /* Cách header một khoảng */
}

.checklist-card {
    background: #ffffff;
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
    border: 1px solid #f0f0f0;
}

.checklist-title {
    margin-top: 0;
    margin-bottom: 15px;
    font-size: 16px;
    color: #2c3e50;
    text-align: center;
    font-weight: 700;
}

.checklist-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(40px, 1fr));
    gap: 10px;
}

.checklist-item {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
    background-color: #f0f4f8;
    color: #7f8c8d;
    font-weight: bold;
    cursor: pointer;
    border: 1px solid #e0e6ed;
    transition: all 0.2s ease;
}

.checklist-item:hover {
    background-color: #e2e8f0;
    transform: scale(1.05);
}

.checklist-item.answered {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-color: transparent;
    box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.checklist-status {
    margin-top: 20px;
    display: flex;
    justify-content: space-around;
    font-size: 13px;
    color: #555;
}

.status-item {
    display: flex;
    align-items: center;
    gap: 6px;
}

.box {
    width: 14px;
    height: 14px;
    border-radius: 3px;
}

.answered-box {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.default-box {
    background-color: #f0f4f8;
    border: 1px solid #e0e6ed;
}

.list {
    background: #ffffff;
    border-radius: 16px;
    padding: 30px;
    margin-bottom: 24px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
    border: 1px solid #f0f0f0;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.list:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
}

.question-container {
    margin-bottom: 20px;
}

.question {
    font-weight: 700;
    font-size: 18px;
    color: #2c3e50;
    line-height: 1.5;
}

.answers-container {
    padding-left: 10px;
}

.radio-card {
    padding: 12px 16px;
    border-radius: 10px;
    border: 1px solid #e0e6ed;
    background-color: #fafbfc;
    cursor: pointer;
    transition: all 0.2s ease;
    display: flex;
    align-items: center;
}

.radio-card:hover {
    background-color: #f0f4f8;
    border-color: #c0cddb;
}

.radio-card.selected {
    background-color: #ebf5ff;
    border-color: #3b82f6;
    box-shadow: 0 2px 8px rgba(59, 130, 246, 0.15);
}

:deep(.n-radio) {
    width: 100%;
}

.btn-end {
    display: flex;
    justify-content: center;
    margin-top: 40px;
}

.submit-btn {
    height: 50px;
    padding: 0 40px;
    font-size: 18px;
    font-weight: bold;
    border-radius: 25px;
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    color: white;
    border: none;
    box-shadow: 0 5px 15px rgba(245, 87, 108, 0.4);
    transition: all 0.3s;
}

.submit-btn:hover {
    transform: scale(1.05);
    box-shadow: 0 8px 25px rgba(245, 87, 108, 0.5);
    color: white;
}
</style>