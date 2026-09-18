<script setup>
import { ref, computed, onBeforeUnmount, defineEmits } from "vue";

const emit = defineEmits(['time-up'])

const { student, time, name }
    = defineProps([
        "student",
        "time",
        "name"
    ])


// xu ly dem nguoc thoi gian neu can

const timeLeftInSeconds = ref(time * 60);

const formattedTime = computed(() =>
{
    const minutes = Math.floor(timeLeftInSeconds.value / 60);
    const seconds = timeLeftInSeconds.value % 60;
    return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
});

const interval = setInterval(() =>
{
    timeLeftInSeconds.value -= 1;
    if (timeLeftInSeconds.value <= 0)
    {
        clearInterval(interval);
        emit('time-up');
    }
}, 1000);

onBeforeUnmount(() =>
{
    clearInterval(interval);
});



</script>
<template>
    <main>
        <div class="header-container">
            <div class="student-info">
                <div class="avatar">
                    {{ student.student_name ? student.student_name.charAt(0).toUpperCase() : student.name.charAt(0).toUpperCase() }}
                </div>
                <div class="details">
                    <span class="name">{{ student.student_name || student.name }}</span>
                    <span class="subtext">{{ student.student_code }} - {{ student.class_name }}</span>
                </div>
            </div>

            <div class="name-test">
                <h2>{{ name }}</h2>
            </div>

            <div class="timer-container" :class="{ 'danger': timeLeftInSeconds <= 60 }">
                <span class="icon">⏱️</span>
                <span class="timex">{{ formattedTime }}</span>
            </div>
        </div>
    </main>
</template>

<style scoped>
.header-container {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    display: flex;
    height: 80px;
    align-items: center;
    padding: 0 40px;
    justify-content: space-between;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
    border-bottom: 1px solid #f0f0f0;
}

.student-info {
    display: flex;
    align-items: center;
    gap: 12px;
    flex: 1;
}

.avatar {
    width: 44px;
    height: 44px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    font-weight: bold;
    box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.details {
    display: flex;
    flex-direction: column;
}

.details .name {
    font-weight: 700;
    color: #2c3e50;
    font-size: 16px;
}

.details .subtext {
    font-size: 13px;
    color: #7f8c8d;
}

.name-test {
    flex: 2;
    text-align: center;
}

.name-test h2 {
    color: #2c3e50;
    font-weight: 700;
    font-size: 22px;
    margin: 0;
    background: linear-gradient(135deg, #2c3e50 0%, #3498db 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}

.timer-container {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 8px;
    font-size: 24px;
    font-weight: 700;
    color: #2c3e50;
}

.timer-container.danger {
    color: #e74c3c;
    animation: pulse 1s infinite;
}

.timer-container .icon {
    font-size: 20px;
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
}
</style>