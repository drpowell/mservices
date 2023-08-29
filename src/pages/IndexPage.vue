<script setup>
import yaml from "js-yaml";
import axios from "axios";
import { onMounted, ref, watch, reactive } from "vue";

import Questions from "../components/QuestionList.vue";
import Services from "../components/ServiceBoxes.vue";
import Comparison from "../components/ComparisonTable.vue";

const spec = ref([]);
const qSelected = reactive({});
const sSelected = ref([]);
const optionHighlight = ref(null);

onMounted(async () => {
    const resp = await axios.get("spec.yaml");
    spec.value = yaml.load(resp.data);
    // console.log("mounted", spec.value);
    qClear();
});

// Initialise all questions to unanswered
const qClear = () => {
    for (let q of spec.value.questions) {
        qSelected[q.name] = [];
    }
};

const setSelected = (v) => {
    sSelected.value = v;
};

const highlightService = (s) => {
    if (s) {
        optionHighlight.value = s.match;
    } else {
        optionHighlight.value = null;
    }
};

// watch(qSelected, () => {
//     console.log("qSelected", qSelected);
// });
// watch(sSelected.value, () => {
//     console.log("sSelected", sSelected);
// });
</script>

<template>
    <q-page class="flex flex-center">
        <div class="row">
            <q-scroll-area class="q-pa-sm col-3" style="max-height: 80vh">
                <div class="desc">
                    <h4>Describe your data</h4>
                    Answer these questions to help identify data storage
                    services that are suitable for your needs. Checking these
                    boxes will change the list of available services. If you are
                    uncertain how to answer, leave the question blank to
                    maximize your resulting options.
                    <q-btn
                        outline
                        size="sm"
                        color="primary"
                        label="Clear Answers"
                        @click="qClear()"
                    />
                </div>
                <hr />
                <Questions
                    :questions="spec.questions"
                    :selected="qSelected"
                    :highlight="optionHighlight"
                    @update:selected="(name, v) => (qSelected[name] = v)"
                />
            </q-scroll-area>
            <div class="col-6 q-mt-xl">
                <Services
                    :services="spec.services"
                    :qSelected="qSelected"
                    @update:model-value="setSelected"
                    @highlight="(z) => highlightService(z)"
                />
            </div>
        </div>
        <div class="row" style="width: 100%">
            <hr />
            <div class="col-12 q-pa-xl">
                <Comparison :services="spec.services" :sSelected="sSelected" />
            </div>
        </div>
    </q-page>
</template>
