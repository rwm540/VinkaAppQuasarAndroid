<template>
  <q-page class="q-pa-md">
    <div class="text-center q-mb-lg">
      <h1 class="text-h4 q-my-md">ضبط صدا و تماس</h1>
      <p class="text-grey-7">اپلیکیشن ضبط مداوم صدا و اتوماتیک تماس‌ها</p>
    </div>

    <!-- بخش ضبط صدا -->
    <q-card class="q-mb-lg">
      <q-card-section>
        <div class="text-h6">ضبط صدا</div>
        <div class="text-caption text-grey-7">ضبط مداوم صدا تا زمان توقف دستی یا خاموش شدن دستگاه</div>
      </q-card-section>

      <q-card-section>
        <div class="row items-center q-gutter-md">
          <q-btn
            :color="isRecording ? 'red' : 'primary'"
            :icon="isRecording ? 'stop' : 'mic'"
            :label="isRecording ? 'توقف ضبط' : 'شروع ضبط'"
            @click="toggleRecording"
            :loading="recordingLoading"
            size="lg"
          />

          <q-chip
            :color="isRecording ? 'red' : 'grey'"
            text-color="white"
            :icon="isRecording ? 'radio_button_checked' : 'radio_button_unchecked'"
          >
            {{ isRecording ? 'در حال ضبط' : 'متوقف' }}
          </q-chip>

          <div class="col">
            <div class="text-caption">مدت زمان ضبط: {{ formatDuration(recordingDuration) }}</div>
          </div>
        </div>
      </q-card-section>
    </q-card>

    <!-- بخش ضبط تماس -->
    <q-card class="q-mb-lg">
      <q-card-section>
        <div class="text-h6">ضبط تماس</div>
        <div class="text-caption text-grey-7">ضبط اتوماتیک تماس‌ها هنگام زنگ خوردن</div>
      </q-card-section>

      <q-card-section>
        <div class="row items-center q-gutter-md">
          <q-toggle
            v-model="autoCallRecording"
            label="ضبط اتوماتیک تماس"
            color="primary"
          />

          <q-chip
            :color="autoCallRecording ? 'green' : 'grey'"
            text-color="white"
          >
            {{ autoCallRecording ? 'فعال' : 'غیرفعال' }}
          </q-chip>

          <q-btn
            color="secondary"
            icon="info"
            label="نکته"
            @click="showCallRecordingInfo"
            flat
            size="sm"
          />
        </div>

        <div v-if="currentCall" class="q-mt-md">
          <q-banner class="bg-orange-1 text-orange-9" rounded>
            <template v-slot:avatar>
              <q-icon name="phone" />
            </template>
            تماس در حال ضبط: {{ currentCall }}
          </q-banner>
        </div>
      </q-card-section>
    </q-card>

    <!-- لیست فایل‌های ضبط شده -->
    <q-card>
      <q-card-section>
        <div class="text-h6">فایل‌های ضبط شده</div>
      </q-card-section>

      <q-list bordered separator>
        <q-item v-for="file in recordedFiles" :key="file.name">
          <q-item-section avatar>
            <q-icon name="audiotrack" />
          </q-item-section>
          <q-item-section>
            <q-item-label>{{ file.name }}</q-item-label>
            <q-item-label caption>{{ formatFileSize(file.size) }} • {{ formatDate(file.date) }}</q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-btn flat round icon="play_arrow" @click="playFile(file)" />
            <q-btn flat round icon="delete" @click="deleteFile(file)" color="negative" />
          </q-item-section>
        </q-item>

        <q-item v-if="recordedFiles.length === 0">
          <q-item-section>
            <q-item-label class="text-grey-6">هیچ فایلی ضبط نشده</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-card>

    <!-- تنظیمات -->
    <q-card class="q-mt-lg">
      <q-card-section>
        <div class="text-h6">تنظیمات</div>
      </q-card-section>

      <q-card-section>
        <q-option-group
          v-model="audioQuality"
          :options="qualityOptions"
          color="primary"
          type="radio"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { Capacitor } from '@capacitor/core';
import { Filesystem, Directory } from '@capacitor/filesystem';

// وضعیت ضبط
const isRecording = ref(false);
const recordingLoading = ref(false);
const recordingDuration = ref(0);
const recordingInterval = ref<NodeJS.Timeout | null>(null);

// تنظیمات ضبط تماس
const autoCallRecording = ref(true);
const currentCall = ref<string | null>(null);

// فایل‌های ضبط شده
interface RecordedFile {
  name: string;
  size: number;
  date: Date;
  path: string;
}

const recordedFiles = ref<RecordedFile[]>([]);

// تنظیمات کیفیت صدا
const audioQuality = ref('high');
const qualityOptions = [
  { label: 'کیفیت بالا (WAV)', value: 'high' },
  { label: 'کیفیت متوسط (MP3)', value: 'medium' },
  { label: 'کیفیت پایین (AAC)', value: 'low' }
];

// متغیرهای ضبط
let mediaRecorder: MediaRecorder | null = null;
let audioChunks: Blob[] = [];
let stream: MediaStream | null = null;

// توابع فرمت
const formatDuration = (seconds: number): string => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
};

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
};

const formatDate = (date: Date): string => {
  return date.toLocaleDateString('fa-IR') + ' ' + date.toLocaleTimeString('fa-IR');
};

// توابع ضبط صدا
const toggleRecording = async () => {
  if (isRecording.value) {
    stopRecording();
  } else {
    await startRecording();
  }
};

const startRecording = async () => {
  try {
    recordingLoading.value = true;

    if (Capacitor.isNativePlatform()) {
      // در پلتفرم‌های native از Capacitor API استفاده می‌کنیم
      // برای سادگی، از Web Audio API استفاده می‌کنیم
      stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    } else {
      // در وب از Web Audio API استفاده می‌کنیم
      stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    }

    mediaRecorder = new MediaRecorder(stream);
    audioChunks = [];

    mediaRecorder.ondataavailable = (event) => {
      audioChunks.push(event.data);
    };

    mediaRecorder.onstop = async () => {
      const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
      await saveRecording(audioBlob);
    };

    mediaRecorder.start();
    isRecording.value = true;
    recordingDuration.value = 0;

    recordingInterval.value = setInterval(() => {
      recordingDuration.value++;
    }, 1000);

  } catch (error) {
    console.error('خطا در شروع ضبط:', error);
    // نمایش پیام خطا به کاربر
  } finally {
    recordingLoading.value = false;
  }
};

const stopRecording = () => {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop();
  }

  if (stream) {
    stream.getTracks().forEach(track => track.stop());
  }

  if (recordingInterval.value) {
    clearInterval(recordingInterval.value);
  }

  isRecording.value = false;
};

const saveRecording = async (blob: Blob) => {
  try {
    const fileName = `recording_${Date.now()}.wav`;

    if (Capacitor.isNativePlatform()) {
      // ذخیره در دستگاه native
      const base64Data = await blobToBase64(blob);
      await Filesystem.writeFile({
        path: fileName,
        data: base64Data,
        directory: Directory.Documents
      });
    } else {
      // ذخیره در وب
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = fileName;
      a.click();
      URL.revokeObjectURL(url);
    }

    // اضافه کردن به لیست فایل‌ها
    recordedFiles.value.unshift({
      name: fileName,
      size: blob.size,
      date: new Date(),
      path: fileName
    });

  } catch (error) {
    console.error('خطا در ذخیره فایل:', error);
  }
};

const blobToBase64 = (blob: Blob): Promise<string> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => {
      const result = reader.result;
      if (typeof result === 'string' && result.includes(',')) {
        const parts = result.split(',');
        if (parts.length > 1 && parts[1]) {
          resolve(parts[1]);
        } else {
          reject(new Error('Invalid data URL format'));
        }
      } else {
        reject(new Error('Failed to read blob as string'));
      }
    };
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
};

// توابع فایل‌ها
const playFile = (file: RecordedFile) => {
  // پیاده‌سازی پخش فایل
  console.log('پخش فایل:', file.name);
};

const deleteFile = async (file: RecordedFile) => {
  try {
    if (Capacitor.isNativePlatform()) {
      await Filesystem.deleteFile({
        path: file.path,
        directory: Directory.Documents
      });
    }

    // حذف از لیست
    const index = recordedFiles.value.findIndex(f => f.name === file.name);
    if (index > -1) {
      recordedFiles.value.splice(index, 1);
    }
  } catch (error) {
    console.error('خطا در حذف فایل:', error);
  }
};

// مدیریت تماس‌ها (در نسخه کامل نیاز به پلاگین سفارشی دارد)
const setupCallRecording = () => {
  // این قسمت نیاز به پلاگین سفارشی Capacitor دارد
  // برای نسخه آزمایشی، شبیه‌سازی می‌کنیم
  console.log('ضبط تماس تنظیم شد');
};

const showCallRecordingInfo = () => {
  // نمایش اطلاعات درباره ضبط تماس
  alert('ضبط تماس نیاز به پلاگین سفارشی Capacitor دارد که با Android API کار کند. در نسخه فعلی، این ویژگی شبیه‌سازی شده است.');
};

// lifecycle
onMounted(() => {
  void setupCallRecording();
  void loadRecordedFiles();
});

onUnmounted(() => {
  if (recordingInterval.value) {
    clearInterval(recordingInterval.value);
  }
  if (isRecording.value) {
    stopRecording();
  }
});

const loadRecordedFiles = async () => {
  // بارگذاری لیست فایل‌های ضبط شده قبلی
  // در نسخه کامل، فایل‌ها را از دستگاه می‌خوانیم
};
</script>

<style scoped>
.q-page {
  max-width: 800px;
  margin: 0 auto;
}
</style>
