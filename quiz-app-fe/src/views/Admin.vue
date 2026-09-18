<script setup>
import { ref, onMounted, h } from 'vue'
import { NButton, NPopconfirm, NSpace, NTag } from 'naive-ui'
import axios from 'axios'

const API_BASE = "http://localhost:8004"
const activeTab = ref('tests')

// Auth state
const isAuthenticated = ref(false)
const username = ref('')
const password = ref('')
const loginError = ref('')

// Dữ liệu
const students = ref([])
const results = ref([])
const questions = ref([])
const tests = ref([])
const selectedTestId = ref(null)
const loading = ref(false)

// CRUD Test state
const showTestModal = ref(false)
const isEditingTest = ref(false)
const editingTestId = ref(null)
const testForm = ref({
    test_code: '',
    name: '',
    time: "45",
    list_students: ''
})

// CRUD Question state
const showQuestionModal = ref(false)
const isEditing = ref(false)
const editingQuestionId = ref(null)
const questionForm = ref({
    text: '',
    answers: [
        { text: '', is_correct: true },
        { text: '', is_correct: false },
        { text: '', is_correct: false },
        { text: '', is_correct: false }
    ]
})

// Columns cho Data Table
const testColumns = [
    { title: 'Mã đề', key: 'test_code' },
    { title: 'Tên bài thi', key: 'name' },
    { title: 'Thời gian', key: 'time', render: (row) => `${row.time} phút` },
    { title: 'Hành động', key: 'actions', render: (row) => {
        return h(NSpace, {}, {
            default: () => [
                h(NButton, { size: 'small', type: 'info', onClick: () => openEditTestModal(row) }, { default: () => 'Sửa' }),
                h(NPopconfirm, {
                    onPositiveClick: () => deleteTest(row._id),
                    positiveText: 'Xóa',
                    negativeText: 'Hủy'
                }, {
                    trigger: () => h(NButton, { size: 'small', type: 'error' }, { default: () => 'Xóa' }),
                    default: () => 'Xóa mã đề sẽ mồ côi các câu hỏi thuộc mã đề này. Xác nhận xóa?'
                })
            ]
        })
    }}
]

const studentColumns = [
    { title: 'Mã Sinh Viên', key: 'student_code' },
    { title: 'Họ và Tên', key: 'student_name' }
]

const resultColumns = [
    { title: 'Mã SV', key: 'student_code' },
    { title: 'Bài Thi (ID)', key: 'test_id' },
    { title: 'Điểm', key: 'score', render: (row) => `${row.score}/10` },
    { title: 'Số câu đúng', key: 'correct_count', render: (row) => `${row.correct_count}/${row.total_questions}` },
    { title: 'Thời gian nộp', key: 'submitted_at', render: (row) => new Date(row.submitted_at).toLocaleString() }
]

const questionColumns = [
    { title: 'Câu hỏi', key: 'text' },
    { title: 'Đáp án đúng', key: 'correct', render: (row) => {
        const correctAns = row.answers.find(a => a.is_correct)
        return correctAns ? h(NTag, { type: 'success' }, { default: () => correctAns.text }) : 'N/A'
    }},
    { title: 'Hành động', key: 'actions', render: (row) => {
        return h(NSpace, {}, {
            default: () => [
                h(NButton, { size: 'small', type: 'info', onClick: () => openEditModal(row) }, { default: () => 'Sửa' }),
                h(NPopconfirm, {
                    onPositiveClick: () => deleteQuestion(row._id),
                    positiveText: 'Xóa',
                    negativeText: 'Hủy'
                }, {
                    trigger: () => h(NButton, { size: 'small', type: 'error' }, { default: () => 'Xóa' }),
                    default: () => 'Bạn có chắc muốn xóa câu hỏi này?'
                })
            ]
        })
    }}
]

function handleLogin() {
    if (username.value === 'admin' && password.value === 'admin') {
        isAuthenticated.value = true
        loginError.value = ''
        fetchData()
    } else {
        loginError.value = 'Sai tài khoản hoặc mật khẩu!'
    }
}

async function fetchData() {
    if (!isAuthenticated.value) return
    loading.value = true
    try {
        if (activeTab.value === 'tests') {
            const res = await axios.get(`${API_BASE}/task_service/admin/tests`)
            if (res.data.status === 'success') {
                tests.value = res.data.data
                if (!selectedTestId.value && tests.value.length > 0) {
                    selectedTestId.value = tests.value[0]._id
                }
            }
        } else if (activeTab.value === 'students') {
            const res = await axios.get(`${API_BASE}/task_service/admin/students`)
            if (res.data.status === 'success') students.value = res.data.data
        } else if (activeTab.value === 'results') {
            const res = await axios.get(`${API_BASE}/task_service/admin/results`)
            if (res.data.status === 'success') results.value = res.data.data
        } else if (activeTab.value === 'questions') {
            // Lấy list tests trước để tạo dropdown
            if (tests.value.length === 0) {
                const resTests = await axios.get(`${API_BASE}/task_service/admin/tests`)
                if (resTests.data.status === 'success') {
                    tests.value = resTests.data.data
                    if (!selectedTestId.value && tests.value.length > 0) {
                        selectedTestId.value = tests.value[0]._id
                    }
                }
            }
            if (selectedTestId.value) {
                const res = await axios.post(`${API_BASE}/task_service/admin/questions`, { test_id: selectedTestId.value })
                if (res.data.status === 'success') questions.value = res.data.data
            }
        }
    } catch (error) {
        console.error("Lỗi lấy dữ liệu Admin:", error)
    } finally {
        loading.value = false
    }
}

// ================= TEST CRUD =================
function openCreateTestModal() {
    isEditingTest.value = false
    editingTestId.value = null
    testForm.value = { test_code: '', name: '', time: "45", list_students: '' }
    showTestModal.value = true
}

function openEditTestModal(t) {
    isEditingTest.value = true
    editingTestId.value = t._id
    testForm.value = {
        test_code: t.test_code || '',
        name: t.name,
        time: String(t.time),
        list_students: (t.list_students || t.list_student || []).join(', ')
    }
    showTestModal.value = true
}

async function saveTest() {
    loading.value = true
    try {
        const studentList = testForm.value.list_students.split(',').map(s => s.trim()).filter(s => s)
        const payload = {
            test_code: testForm.value.test_code,
            name: testForm.value.name,
            time: parseInt(testForm.value.time) || 45,
            list_students: studentList
        }
        if (isEditingTest.value) {
            await axios.put(`${API_BASE}/task_service/admin/tests/update`, { ...payload, test_id: editingTestId.value })
        } else {
            await axios.post(`${API_BASE}/task_service/admin/tests/create`, payload)
        }
        showTestModal.value = false
        fetchData()
    } catch (error) {
        console.error("Lỗi lưu test:", error)
    } finally {
        loading.value = false
    }
}

async function deleteTest(id) {
    loading.value = true
    try {
        await axios.delete(`${API_BASE}/task_service/admin/tests/delete`, { data: { test_id: id } })
        fetchData()
    } catch (error) {
        console.error("Lỗi xóa test:", error)
    } finally {
        loading.value = false
    }
}

// ================= QUESTION CRUD =================
function openCreateModal() {
    if (!selectedTestId.value) {
        alert("Vui lòng chọn một mã đề trước!")
        return
    }
    isEditing.value = false
    editingQuestionId.value = null
    questionForm.value = {
        text: '',
        answers: [
            { text: '', is_correct: true },
            { text: '', is_correct: false },
            { text: '', is_correct: false },
            { text: '', is_correct: false }
        ]
    }
    showQuestionModal.value = true
}

function openEditModal(question) {
    isEditing.value = true
    editingQuestionId.value = question._id
    questionForm.value = {
        text: question.text,
        answers: JSON.parse(JSON.stringify(question.answers)) // deep copy
    }
    showQuestionModal.value = true
}

function setCorrectAnswer(index) {
    questionForm.value.answers.forEach((ans, i) => {
        ans.is_correct = (i === index)
    })
}

async function saveQuestion() {
    loading.value = true
    try {
        if (isEditing.value) {
            await axios.put(`${API_BASE}/task_service/admin/questions/update`, {
                question_id: editingQuestionId.value,
                text: questionForm.value.text,
                answers: questionForm.value.answers
            })
        } else {
            await axios.post(`${API_BASE}/task_service/admin/questions/create`, {
                test_id: selectedTestId.value,
                text: questionForm.value.text,
                answers: questionForm.value.answers
            })
        }
        showQuestionModal.value = false
        fetchData()
    } catch (error) {
        console.error("Lỗi lưu câu hỏi:", error)
    } finally {
        loading.value = false
    }
}

async function deleteQuestion(id) {
    loading.value = true
    try {
        await axios.delete(`${API_BASE}/task_service/admin/questions/delete`, {
            data: { question_id: id }
        })
        fetchData()
    } catch (error) {
        console.error("Lỗi xóa câu hỏi:", error)
    } finally {
        loading.value = false
    }
}

onMounted(() => {
    if (isAuthenticated.value) fetchData()
})
</script>

<template>
    <div class="admin-wrapper">
        <transition name="fade" mode="out-in">
            <div v-if="!isAuthenticated" class="login-container" key="login">
                <div class="glass-card">
                    <h2 class="title">Đăng nhập Admin</h2>
                    <p class="subtitle">Hệ thống Quản trị Quiz App</p>
                    
                    <n-alert v-if="loginError" type="error" style="margin-bottom: 20px;">
                        {{ loginError }}
                    </n-alert>

                    <div class="input-group">
                        <n-input v-model:value="username" size="large" placeholder="Tài khoản" @keyup.enter="handleLogin" />
                    </div>
                    <div class="input-group">
                        <n-input v-model:value="password" size="large" type="password" show-password-on="click" placeholder="Mật khẩu" @keyup.enter="handleLogin" />
                    </div>

                    <n-button @click="handleLogin" class="submit-btn" type="primary" size="large" block>
                        Đăng Nhập
                    </n-button>
                    
                    <div style="text-align: center; margin-top: 15px;">
                        <router-link to="/" style="color: #fff; text-decoration: none;">&larr; Về trang chủ</router-link>
                    </div>
                </div>
            </div>

            <div v-else class="admin-layout" key="dashboard">
                <n-layout has-sider style="height: 100vh;">
                    <n-layout-sider bordered collapse-mode="width" :collapsed-width="64" :width="240"
                        :native-scrollbar="false" class="sidebar">
                        <div class="logo">
                            <h2>Admin Panel</h2>
                        </div>
                        <n-menu :value="activeTab" @update:value="val => { activeTab = val; fetchData(); }" :options="[
                            { label: 'Quản lý Mã Đề', key: 'tests' },
                            { label: 'Ngân hàng câu hỏi', key: 'questions' },
                            { label: 'Kết quả thi', key: 'results' },
                            { label: 'Danh sách sinh viên', key: 'students' }
                        ]" />
                        <div style="padding: 20px; position: absolute; bottom: 0;">
                            <router-link to="/">
                                <n-button type="info" ghost>Về trang chủ</n-button>
                            </router-link>
                        </div>
                    </n-layout-sider>

                    <n-layout>
                        <n-layout-header bordered class="header">
                            <h2>
                                <span v-if="activeTab === 'tests'">Quản lý Mã Đề Thi</span>
                                <span v-else-if="activeTab === 'results'">Quản lý Kết quả thi</span>
                                <span v-else-if="activeTab === 'students'">Danh sách Sinh viên</span>
                                <span v-else>Ngân hàng Câu hỏi</span>
                            </h2>
                            <div>
                                <n-button v-if="activeTab === 'tests'" @click="openCreateTestModal" type="info" style="margin-right: 10px;">+ Thêm Mã Đề</n-button>
                                <n-button v-if="activeTab === 'questions'" @click="openCreateModal" type="info" style="margin-right: 10px;">+ Thêm Câu Hỏi</n-button>
                                <n-button @click="fetchData" :loading="loading" type="primary" style="margin-right: 10px;">Làm mới</n-button>
                                <n-button @click="isAuthenticated = false" type="error" ghost>Đăng xuất</n-button>
                            </div>
                        </n-layout-header>

                        <n-layout-content content-style="padding: 24px; background: #f5f7fa; min-height: calc(100vh - 64px);">
                            <n-card v-if="activeTab === 'tests'">
                                <n-data-table :columns="testColumns" :data="tests" :loading="loading" :bordered="false" />
                            </n-card>

                            <n-card v-if="activeTab === 'results'">
                                <n-data-table :columns="resultColumns" :data="results" :loading="loading" :bordered="false" />
                            </n-card>

                            <n-card v-if="activeTab === 'students'">
                                <n-data-table :columns="studentColumns" :data="students" :loading="loading" :bordered="false" />
                            </n-card>

                            <n-card v-if="activeTab === 'questions'">
                                <div style="margin-bottom: 20px; display: flex; align-items: center; gap: 15px;">
                                    <strong>Chọn Mã Đề:</strong>
                                    <select v-model="selectedTestId" @change="fetchData" style="padding: 8px; border-radius: 6px; border: 1px solid #ccc; width: 300px;">
                                        <option v-for="t in tests" :key="t._id" :value="t._id">{{ t.test_code }} - {{ t.name }}</option>
                                    </select>
                                </div>
                                <n-data-table :columns="questionColumns" :data="questions" :loading="loading" :bordered="false" />
                            </n-card>
                        </n-layout-content>
                    </n-layout>
                </n-layout>

                <!-- Modal Thêm/Sửa Mã Đề -->
                <n-modal v-model:show="showTestModal" preset="card" style="width: 500px;" :title="isEditingTest ? 'Sửa Mã Đề' : 'Thêm Mã Đề Mới'">
                    <n-space vertical size="large">
                        <div>
                            <label style="font-weight: bold;">Mã đề (VD: SOA_01):</label>
                            <n-input v-model:value="testForm.test_code" placeholder="Mã đề viết liền không dấu" />
                        </div>
                        <div>
                            <label style="font-weight: bold;">Tên bài thi:</label>
                            <n-input v-model:value="testForm.name" placeholder="Tên bài kiểm tra" />
                        </div>
                        <div>
                            <label style="font-weight: bold;">Thời gian (phút):</label>
                            <n-input v-model:value="testForm.time" type="text" placeholder="Ví dụ: 45" />
                        </div>
                        <div>
                            <label style="font-weight: bold;">Danh sách MSV (cách nhau dấu phẩy):</label>
                            <n-input v-model:value="testForm.list_students" type="textarea" placeholder="2311060738, 2311060001 (để trống nếu cho phép tất cả)" />
                        </div>
                        <div style="display: flex; justify-content: flex-end; gap: 10px; margin-top: 20px;">
                            <n-button @click="showTestModal = false">Hủy</n-button>
                            <n-button type="primary" @click="saveTest" :loading="loading">Lưu</n-button>
                        </div>
                    </n-space>
                </n-modal>

                <!-- Modal Thêm/Sửa Câu Hỏi -->
                <n-modal v-model:show="showQuestionModal" preset="card" style="width: 600px;" :title="isEditing ? 'Sửa Câu Hỏi' : 'Thêm Câu Hỏi Mới'">
                    <n-space vertical size="large">
                        <div>
                            <label style="font-weight: bold;">Nội dung câu hỏi:</label>
                            <n-input v-model:value="questionForm.text" type="textarea" placeholder="Nhập nội dung câu hỏi..." />
                        </div>
                        <div>
                            <label style="font-weight: bold;">Các đáp án (Chọn 1 đáp án đúng):</label>
                            <div v-for="(ans, index) in questionForm.answers" :key="index" style="display: flex; align-items: center; gap: 10px; margin-top: 10px;">
                                <input type="radio" :name="'correct_answer'" :checked="ans.is_correct" @change="setCorrectAnswer(index)" style="width: 20px; height: 20px; cursor: pointer;" />
                                <n-input v-model:value="ans.text" :placeholder="`Đáp án ${String.fromCharCode(65 + index)}`" style="flex: 1;" />
                            </div>
                        </div>
                        <div style="display: flex; justify-content: flex-end; gap: 10px; margin-top: 20px;">
                            <n-button @click="showQuestionModal = false">Hủy</n-button>
                            <n-button type="primary" @click="saveQuestion" :loading="loading">Lưu</n-button>
                        </div>
                    </n-space>
                </n-modal>
            </div>
        </transition>
    </div>
</template>

<style scoped>
.admin-wrapper {
    height: 100vh;
    width: 100vw;
}

/* Auth Styles */
.login-container {
    height: 100vh;
    width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(-45deg, #1a2a6c, #11998e, #38ef7d, #b21f1f);
    background-size: 400% 400%;
    animation: gradientBG 15s ease infinite;
}

@keyframes gradientBG {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

.glass-card {
    background: rgba(255, 255, 255, 0.15);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.18);
    padding: 40px;
    width: 90%;
    max-width: 400px;
}

.title {
    color: #fff;
    font-size: 28px;
    font-weight: 700;
    margin-bottom: 8px;
    text-align: center;
}

.subtitle {
    color: rgba(255, 255, 255, 0.9);
    text-align: center;
    margin-bottom: 30px;
}

.input-group {
    margin-bottom: 16px;
}

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

/* Dashboard Styles */
.admin-layout {
    height: 100vh;
    width: 100vw;
}

.sidebar {
    background: #ffffff;
}

.logo {
    height: 64px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-bottom: 1px solid #efeff5;
    color: #333;
}

.logo h2 {
    margin: 0;
    font-size: 20px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}

.header {
    height: 64px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 24px;
    background: #fff;
}

.header h2 {
    margin: 0;
    font-size: 20px;
    font-weight: 500;
    color: #333;
}

:deep(.n-card) {
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

/* Transitions */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>
