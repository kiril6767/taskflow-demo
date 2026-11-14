
<script setup>
import { computed, reactive, ref } from 'vue'

const props = defineProps({
  task: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(['save', 'cancel'])

const form = reactive({
  title: props.task?.title ?? '',
  description: props.task?.description ?? '',
  status: props.task?.status ?? 'todo',
  priority: props.task?.priority ?? 'medium',
  dueDate: props.task?.dueDate ?? '',
})

const showErrors = ref(false)
const isEditing = computed(() => props.task !== null)

const statusLabels = {
  todo: 'Очікує',
  progress: 'У процесі',
  done: 'Виконано',
}

const priorityLabels = {
  low: 'Низький',
  medium: 'Середній',
  high: 'Високий',
}

function isValidDate(value) {
  if (!/^\d{4}-\d{2}-\d{2}$/.test(value)) return false

  const date = new Date(`${value}T12:00:00`)

  return !Number.isNaN(date.getTime()) &&
    `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}` === value
}

const errors = computed(() => {
  const result = {}

  if (!form.title.trim()) {
    result.title = 'Введіть назву завдання'
  } else if (form.title.trim().length < 3) {
    result.title = 'Назва має містити щонайменше 3 символи'
  } else if (form.title.trim().length > 80) {
    result.title = 'Назва не може містити більше 80 символів'
  }

  if (form.description.trim().length > 300) {
    result.description = 'Опис не може містити більше 300 символів'
  }

  if (!['todo', 'progress', 'done'].includes(form.status)) {
    result.status = 'Оберіть правильний статус'
  }

  if (!['low', 'medium', 'high'].includes(form.priority)) {
    result.priority = 'Оберіть правильний пріоритет'
  }

  if (!isValidDate(form.dueDate)) {
    result.dueDate = 'Оберіть коректну дату виконання'
  }

  return result
})

function submitForm() {
  showErrors.value = true

  if (Object.keys(errors.value).length > 0) return

  emit('save', {
    title: form.title.trim(),
    description: form.description.trim(),
    status: form.status,
    priority: form.priority,
    dueDate: form.dueDate,
  })
}
</script>

<template>
  <section class="task-form-panel">
    <h2>{{ isEditing ? 'Редагувати завдання' : 'Нове завдання' }}</h2>

    <div class="task-preview">
      <h3>Попередній перегляд</h3>
      <strong>{{ form.title || 'Назва завдання' }}</strong>
      <p>{{ form.description || 'Опис завдання з’явиться тут' }}</p>
      <span>{{ statusLabels[form.status] }}</span>
      <span> · {{ priorityLabels[form.priority] }}</span>
      <p>Дата виконання: {{ form.dueDate || 'Не вибрана' }}</p>
    </div>

    <form novalidate @submit.prevent="submitForm">
      <div class="form-field">
        <label for="task-title">Назва завдання</label>
        <input
          id="task-title"
          v-model.trim="form.title"
          type="text"
          maxlength="80"
          aria-describedby="title-error"
        />
        <p v-if="showErrors && errors.title" id="title-error" class="field-error">
          {{ errors.title }}
        </p>
      </div>

      <div class="form-field">
        <label for="task-description">Опис</label>
        <textarea
          id="task-description"
          v-model.trim="form.description"
          maxlength="300"
          rows="3"
          aria-describedby="description-error"
        ></textarea>
        <p v-if="showErrors && errors.description" id="description-error" class="field-error">
          {{ errors.description }}
        </p>
      </div>

      <div class="form-field">
        <label for="task-status">Статус</label>
        <select id="task-status" v-model="form.status">
          <option value="todo">Очікує</option>
          <option value="progress">У процесі</option>
          <option value="done">Виконано</option>
        </select>
      </div>

      <div class="form-field">
        <label for="task-priority">Пріоритет</label>
        <select id="task-priority" v-model="form.priority">
          <option value="low">Низький</option>
          <option value="medium">Середній</option>
          <option value="high">Високий</option>
        </select>
      </div>

      <div class="form-field">
        <label for="task-date">Дата виконання</label>
        <input
          id="task-date"
          v-model="form.dueDate"
          type="date"
          aria-describedby="date-error"
        />
        <p v-if="showErrors && errors.dueDate" id="date-error" class="field-error">
          {{ errors.dueDate }}
        </p>
      </div>

      <div class="form-actions">
        <button type="submit">
          {{ isEditing ? 'Зберегти зміни' : 'Створити завдання' }}
        </button>
        <button type="button" @click="emit('cancel')">
          Скасувати
        </button>
      </div>
    </form>
  </section>
</template>
