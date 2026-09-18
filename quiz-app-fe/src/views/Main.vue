<script setup>
import { ref } from "vue"
import axios from "axios";
import ExamsHeader from '@/components/ExamsHeader.vue';
import Question from "../components/Question.vue";
import Result from "../components/Result.vue";


const questions = ref([])
const student = ref({})
const test = ref({})
const time = ref(0)
const show = ref(false)
const alert = ref(false)
const showResult = ref(false)
const examResult = ref(null)
const submitted = ref(false)

const email = ref("")
const password = ref("")
const fullName = ref("")
const className = ref("")
const testCode = ref("")

const wait = ref(false)
const status_text = ref("Đang xử lý...")
const messageAlert = ref("")

// Lưu đáp án sinh viên chọn
const studentAnswers = ref([])

const API_BASE = "http://localhost:8004"

function onAnswersUpdate(answers)
{
    studentAnswers.value = answers
}

async function handleSubmit()
{
    if (submitted.value) return
    submitted.value = true

    try
    {
        const studentCode = email.value.toLowerCase().split("@")[0]
        status_text.value = "Đang nộp bài..."
        wait.value = true
        show.value = false

        const response = await axios.post(`${API_BASE}/task_service/submit_test`, {
            "test_id": test.value._id,
            "student_code": studentCode,
            "answers": studentAnswers.value
        });

        if (response.status === 200 && response.data.status === "success")
        {
            examResult.value = response.data.data
            showResult.value = true
            wait.value = false
        }
        else
        {
            alert.value = true
            messageAlert.value = response.data.message || "Có lỗi khi nộp bài!"
            wait.value = false
            submitted.value = false
        }
    }
    catch (error)
    {
        console.error('Lỗi nộp bài:', error)
        alert.value = true
        messageAlert.value = "Có lỗi kết nối khi nộp bài!"
        wait.value = false
        submitted.value = false
    }
}

function onTimeUp()
{
    // Tự động nộp bài khi hết giờ
    if (!submitted.value)
    {
        handleSubmit()
    }
}

const handleClick = async () =>
{
    // Validate trước khi gửi
    if (!testCode.value || !email.value || !password.value || !fullName.value || !className.value)
    {
        alert.value = true;
        messageAlert.value = "Vui lòng nhập đầy đủ thông tin!";
        return;
    }

    if (!email.value.toLowerCase().endsWith("@hunre.edu.vn"))
    {
        alert.value = true;
        messageAlert.value = "Email phải có đuôi @hunre.edu.vn (ví dụ: 2311060738@hunre.edu.vn)";
        return;
    }

    try
    {
        wait.value = true;
        alert.value = false;

        const studentCode = email.value.toLowerCase().split("@")[0];
        const socket = new WebSocket(`ws://localhost:8004/task_service/ws/status/${studentCode}`);

        socket.onmessage = (event) =>
        {
            console.log('Dữ liệu nhận được:', event.data);
            status_text.value = event.data;
        }

        const response = await axios.post(`${API_BASE}/task_service/start_test`, {
            "test_code": testCode.value,
            "email": email.value.toLowerCase(),
            "password": password.value,
            "full_name": fullName.value,
            "class_name": className.value
        });

        console.log('Trả về:', response.data);

        if (response.status === 200)
        {

            if (response.data.status == 'failed')
            {
                wait.value = false;
                alert.value = true;
                messageAlert.value = response.data.message || "Có lỗi xảy ra!";
                return;
            }
            show.value = true
            wait.value = false
            console.log('Dữ liệu:', response.data);
            questions.value = response.data.data.questions;
            student.value = response.data.data.student;
            test.value = response.data.data.test;
            time.value = test.value.time;
            alert.value = false;
        }
        else
        {
            console.error('Có lỗi xảy ra khi gửi dữ liệu');
            wait.value = false;
            status_text.value = "";
        }
    }
    catch (error)
    {
        console.error('Lỗi:', error);
        wait.value = false;
        alert.value = true;
        messageAlert.value = "Có lỗi kết nối đến server!";
    }
}


</script>

<template>
    <div class="main-wrapper">
        <n-space vertical :size="12" class="alert-container" v-if="alert">
            <n-alert title="Lỗi" type="error" closable @close="alert = false">
                {{ messageAlert }}
            </n-alert>
        </n-space>

        <!-- Nút Admin -->
        <router-link to="/admin" class="admin-link" v-if="!show && !showResult">
            <n-button type="default" ghost circle>
                <template #icon>
                    <span>⚙️</span>
                </template>
            </n-button>
        </router-link>

        <!-- Hiển thị kết quả sau khi nộp bài -->
        <transition name="fade" mode="out-in">
            <div v-if="showResult && examResult" key="result">
                <Result :result="examResult" />
            </div>

            <!-- Hiển thị bài thi -->
            <div v-else-if="show && questions.length > 0" class="exam-view" key="exam">
                <div class="sticky-header">
                    <ExamsHeader :student="student" :time="time" :name="test.name" @time-up="onTimeUp" />
                </div>
                <div class="question-container">
                    <Question :questions="questions" @update:answers="onAnswersUpdate" @submit="handleSubmit" />
                </div>
            </div>

            <!-- Form đăng nhập -->
            <div class="container-main" v-else key="login">
                <div class="glass-card">
                    <div v-if="wait" class="card-spinner">
                        <NSpin size="large" stroke="#667eea" />
                        <p class="loading-text">{{ status_text }}</p>
                    </div>
                    <div class="login-form" v-else>
                        <h2 class="title">Đăng nhập thi trắc nghiệm</h2>
                        <p class="subtitle">Hệ thống thi trực tuyến HUNRE</p>

                        <div class="input-group">
                            <n-input v-model:value="testCode" size="large" type="text" placeholder="Mã đề thi (VD: SOA_01)" />
                        </div>
                        <div class="input-group">
                            <n-input v-model:value="email" size="large" type="text" placeholder="Email (VD: 2311060738@hunre.edu.vn)" />
                        </div>
                        <div class="input-group">
                            <n-input v-model:value="password" size="large" type="password" show-password-on="click" placeholder="Mật khẩu (Mã sinh viên)" />
                        </div>
                        <div class="input-group">
                            <n-input v-model:value="fullName" size="large" type="text" placeholder="Họ và Tên" />
                        </div>
                        <div class="input-group">
                            <n-input v-model:value="className" size="large" type="text" placeholder="Lớp (VD: ĐH CNTT K23A)" />
                        </div>

                        <n-button @click="handleClick" class="submit-btn" type="primary" size="large" block>
                            Bắt đầu làm bài
                        </n-button>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<style scoped>
/* Gradient Animation Background */
.main-wrapper {
    min-height: 100vh;
    width: 100vw;
    background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
    background-size: 400% 400%;
    animation: gradientBG 15s ease infinite;
    position: relative;
    overflow-x: hidden;
}

@keyframes gradientBG {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

.alert-container {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 9999;
    width: 90%;
    max-width: 500px;
}

.admin-link {
    position: absolute;
    top: 20px;
    right: 20px;
    z-index: 100;
}

.container-main {
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
}

/* Glassmorphism Card */
.glass-card {
    background: rgba(255, 255, 255, 0.25);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.18);
    padding: 40px;
    width: 90%;
    max-width: 480px;
    transition: transform 0.3s ease;
}

.glass-card:hover {
    transform: translateY(-5px);
}

.title {
    color: #fff;
    font-size: 28px;
    font-weight: 700;
    margin-bottom: 8px;
    text-align: center;
    text-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.subtitle {
    color: rgba(255, 255, 255, 0.9);
    text-align: center;
    margin-bottom: 30px;
    font-size: 16px;
}

.input-group {
    margin-bottom: 16px;
}

/* Tùy chỉnh input cho Naive UI trong suốt */
:deep(.n-input) {
    background-color: rgba(255, 255, 255, 0.9) !important;
    border-radius: 10px;
}

.submit-btn {
    margin-top: 10px;
    height: 50px;
    font-size: 18px;
    font-weight: bold;
    border-radius: 10px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border: none;
    transition: all 0.3s;
}

.submit-btn:hover {
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
    transform: scale(1.02);
}

.card-spinner {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    height: 300px;
}

.loading-text {
    color: #fff;
    margin-top: 20px;
    font-size: 18px;
    font-weight: 500;
}

/* Exam View */
.exam-view {
    background-color: #f5f7fa;
    min-height: 100vh;
}

.sticky-header {
    position: sticky;
    top: 0;
    z-index: 1000;
}

.question-container {
    padding: 20px;
    max-width: 1000px;
    margin: 0 auto;
}

/* Transition effects */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>