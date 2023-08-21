<script setup>
import { ref, watch } from "vue";

const props = defineProps({
    question: Object,
    optionsSelected: Array,
    optionHighlight: Array,
});
const emit = defineEmits(["update:optionsSelected"]);

const selected = ref(null);
const help = ref(false);

// If it is "radio" control type.  Remove other selections
const change = (idx) => {
    if (props.question.type == "radio") {
        selected.value = selected.value.filter((x) => x == idx);
    }
    // console.log(idx, selected.value);
    emit("update:optionsSelected", selected.value);
};
watch(
    () => props.optionsSelected,
    () => {
        // console.log("QuestionOption modeValue change", props.modelValue);
        selected.value = [...props.optionsSelected];
    },
    { immediate: true }
);
// watch(
//     () => props.optionHighlight,
//     () => {
//         console.log("highlighting", props.optionHighlight);
//     }
// );
</script>

<template>
    <div class="question">
        {{ question.question }}
        <q-icon
            size="sm"
            name="o_info"
            v-if="question.help"
            @click="help = !help"
        />
    </div>
    <q-slide-transition>
        <div class="help" v-show="help" v-html="question.help"></div>
    </q-slide-transition>
    <div>
        <fieldset>
            <label
                v-for="(o, idx) in question.options"
                :key="idx"
                class="option"
                :class="optionHighlight?.includes(idx) ? 'highlight' : ''"
            >
                <input
                    type="checkbox"
                    :value="idx"
                    v-model="selected"
                    @change="change(idx)"
                />
                {{ o }}
            </label>
        </fieldset>
    </div>
</template>

<style scoped>
.question {
    font-family: "Roboto Condensed", "Helvetica Neue", Helvetica, Arial,
        sans-serif;
    font-weight: bold;
    font-size: 18px;
    line-height: 1.2;
    color: rgb(32, 125, 147);
    margin-bottom: 0.25em;
}
.option {
    display: block;
    font-family: "Roboto", "Helvetica Neue", Helvetica, Arial, sans-serif;
    font-weight: normal;
    font-size: 13px;
    color: #333;
}
fieldset {
    border: none;
}
input {
    float: left;
    margin-left: -20px !important;
}
label {
    margin-left: 2em;
}
.highlight {
    background-color: rgb(207, 238, 246);
}
</style>
