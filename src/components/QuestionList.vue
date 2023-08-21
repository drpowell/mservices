<script setup>
import { watch } from "vue";

import Question from "./QuestionOption.vue";

const props = defineProps({
    questions: Array,
    selected: Object,
    highlight: Object,
});
const emit = defineEmits(["update:selected"]);

const updateModel = (name, v) => {
    // console.log("QuestionList updateModel", v);
    emit("update:selected", name, v);
};
</script>

<template>
    <ol>
        <li v-for="q in questions" :key="q.name">
            <Question
                :question="q"
                :options-selected="props.selected[q.name]"
                :optionHighlight="highlight?.[q.name]"
                @update:options-selected="(v) => updateModel(q.name, v)"
            />
        </li>
    </ol>
</template>

<style scoped>
li {
    margin-top: 1em;
}
</style>
