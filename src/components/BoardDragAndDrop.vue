<script setup lang="ts">
import { cloneDeep } from "lodash-es";
import { reactive, watch, toRaw } from "vue";
import type { Board, Column, Task } from "@/types";
import { v4 as uuidv4 } from "uuid";
import draggable from "vuedraggable";

import { useAlerts } from "@/stores/alerts";
const alerts = useAlerts();

const props = defineProps<{
  board: Board;
  tasks: Task[];
  addTask(task: Partial<Task>): Task;
}>();

const emit = defineEmits<{
  (e: "update", payload: Partial<Board>): void;
}>();

const tasks = reactive(cloneDeep(props.tasks));
const board = reactive(cloneDeep(props.board));
const columns = reactive<Column[]>(JSON.parse(board.order as string));

const addColumn = () => {
  columns.push({
    id: uuidv4(),
    title: "New Column",
    taskIds: [],
  });
};

watch(columns, () => {
  emit(
    "update",
    cloneDeep({ ...board, order: JSON.stringify(toRaw(columns)) })
  );
});

const addTask = async ({
  column,
  title,
}: {
  column: Column;
  title: string;
}) => {
  const newTask = { title };
  try {
    const savedTask = await props.addTask(newTask);
    tasks.push({ ...savedTask });
    column.taskIds.push(savedTask.id);
  } catch (error) {
    alerts.error("Error creating task!");
  }
};
</script>

<template>
  <div class="flex items-start py-12">
    <draggable
      :list="columns"
      group="columns"
      item-key="id"
      class="flex flex-grow-0 flex-shrink-0 overflow-x-scroll"
    >
      <template #item="{ element: column }">
        <div
          class="column bg-gray-100 flex flex-col justify-between px-3 py-3 rounded mr-4 w-[300px]"
        >
          <div>
            <h3>{{ column.title }}</h3>
            <draggable
              :list="column.taskIds"
              group="tasks"
              item-key="uid"
              :animation="200"
              ghost-class="ghost-card"
              class="min-h-[400px]"
            >
              <template #item="{ element: taskId }">
                <TaskCard
                  v-if="tasks.find((t: Task) => t.id===taskId)"
                  :task="(tasks.find((t: Task) => t.id===taskId) as Task)"
                  class="mt-3 cursor-move"
                />
              </template>
            </draggable>
            <TaskCreator
              @create="
                addTask({
                  column,
                  title: $event,
                })
              "
            />
          </div>
        </div>
      </template>
    </draggable>
    <button class="text-gray-500">New Column +</button>
  </div>
</template>
