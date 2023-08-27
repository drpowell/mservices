<script setup>
import yaml from "js-yaml";
import axios from "axios";

import { onMounted, ref, watch, reactive } from "vue";
import { exportFile } from "quasar";

const spec = ref([]);
const showYAML = ref(false);

onMounted(async () => {
    const resp = await axios.get("spec.yaml");
    spec.value = yaml.load(resp.data);
    for (const k of ["questions", "fields", "services"]) {
        if (!spec.value[k]) spec.value[k] = [];
    }
    setupQuestions();
});

const qCols = ref([]);
const qRows = ref([]);
const setupQuestions = () => {
    qCols.value = [
        { label: "Description", field: "question" },
        { label: "help", field: "help" },
    ];
    qRows.value = spec.value.questions;
};

const fieldEditor = ref({ service: null, field: null, show: false });
const editField = (service, field) => {
    // console.log("editField", service, field);
    if (!service.fields) service.fields = {};
    if (!service.fields[field.name]) service.fields[field.name] = "";
    fieldEditor.value = { service: service, field: field, show: true };
};

const qMatchEditor = ref({ service: null, question: null, show: false });
const editMatch = (service, question) => {
    // console.log("editMatch", service, question);
    if (!service.match) service.match = {};
    if (!service.match[question.name]) service.match[question.name] = [];
    qMatchEditor.value = { service: service, question: question, show: true };
};
</script>

<template>
    <div class="q-ma-xl">
        <q-dialog v-model="fieldEditor.show">
            <q-card style="min-width: 350px">
                <q-card-section>
                    <div class="text-h6">
                        {{ fieldEditor.service.name }} :
                        {{ fieldEditor.field.name }}
                    </div>
                </q-card-section>

                <q-card-section class="q-pt-none">
                    <q-input
                        type="textarea"
                        rows="5"
                        cols="40"
                        dense
                        v-model="
                            fieldEditor.service.fields[fieldEditor.field.name]
                        "
                        autofocus
                        @keyup.enter.stop
                    />
                </q-card-section>

                <q-card-actions align="right" class="text-primary">
                    <q-btn flat label="Cancel" v-close-popup />
                    <q-btn flat label="Set" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <q-dialog v-model="qMatchEditor.show">
            <q-card style="min-width: 350px">
                <q-card-section>
                    <div class="text-h6">
                        {{ qMatchEditor.service.name }} :
                        {{ qMatchEditor.question.name }}
                    </div>
                </q-card-section>

                <q-card-section class="q-pt-none">
                    <q-select
                        filled
                        v-model="
                            qMatchEditor.service.match[
                                qMatchEditor.question.name
                            ]
                        "
                        multiple
                        :options="qMatchEditor.question.options"
                        label="Select matching options"
                        style="width: 250px"
                    />
                </q-card-section>

                <q-card-actions align="right" class="text-primary">
                    <q-btn flat label="Cancel" v-close-popup />
                    <q-btn flat label="Set" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <q-btn
            @click="exportFile('spec.yaml', yaml.dump(spec))"
            label="Download YAML"
            color="primary"
        />
        <h4>Questions</h4>
        <div class="q-ma-xl">
            <q-markup-table wrap-cells>
                <thead>
                    <tr>
                        <th class="hdr">Name</th>
                        <th class="hdr">Type</th>
                        <th class="hdr">Question</th>
                        <th class="hdr">Options</th>
                        <th class="hdr">Help</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(q, i) in spec.questions" :key="i">
                        <td>
                            <q-input
                                v-model="q.name"
                                dense
                                outlined
                                size="10"
                                style="width: 10em"
                            />
                        </td>
                        <td>
                            <q-select
                                outlined
                                dense
                                options-dense
                                v-model="q.type"
                                :options="['radio', 'checkbox']"
                            />
                        </td>
                        <td>
                            {{ q.question }}
                            <q-popup-edit
                                v-model="q.question"
                                auto-save
                                v-slot="scope"
                                buttons
                            >
                                <q-input
                                    size="60"
                                    v-model="scope.value"
                                    dense
                                    autofocus
                                    @keyup.enter="scope.set"
                                />
                            </q-popup-edit>
                        </td>
                        <td>{{ q.options }}</td>
                        <td>
                            {{ q.help }}
                            <q-popup-edit
                                v-model="q.help"
                                auto-save
                                v-slot="scope"
                                buttons
                            >
                                <q-input
                                    rows="10"
                                    cols="60"
                                    type="textarea"
                                    v-model="scope.value"
                                    dense
                                    autofocus
                                    @keyup.enter.stop
                                />
                            </q-popup-edit>
                        </td>
                        <td>
                            <q-btn
                                round
                                color="primary"
                                icon="delete"
                                @click="() => spec.questions.splice(i, 1)"
                            />
                        </td>
                    </tr>
                </tbody>
            </q-markup-table>
            <q-btn
                color="primary"
                @click="() => spec.questions.push({})"
                label="Add Question"
            />
        </div>
        <h4>Fields</h4>
        <div class="q-ma-xl">
            <q-markup-table wrap-cells>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Description</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(q, i) in spec.fields" :key="i">
                        <td>
                            <q-input
                                v-model="q.name"
                                dense
                                outlined
                                size="10"
                                style="width: 10em"
                            />
                        </td>
                        <td>
                            {{ q.description }}
                            <q-popup-edit
                                v-model="q.description"
                                auto-save
                                v-slot="scope"
                                buttons
                            >
                                <q-input
                                    size="60"
                                    v-model="scope.value"
                                    dense
                                    autofocus
                                    @keyup.enter="scope.set"
                                />
                            </q-popup-edit>
                        </td>
                        <td>
                            <q-btn
                                round
                                color="primary"
                                icon="delete"
                                @click="() => spec.fields.splice(i, 1)"
                            />
                        </td>
                    </tr>
                </tbody>
            </q-markup-table>
            <q-btn
                color="primary"
                @click="() => spec.fields.push({})"
                label="Add Field"
            />
        </div>

        <h4>Services</h4>
        <q-markup-table class="q-ma-xl" wrap-cells>
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Description</th>
                    <th>Question Matches</th>
                    <th style="width: 150px">Fields</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(q, i) in spec.services" :key="i">
                    <td>
                        {{ q.name }}
                        <q-popup-edit
                            v-model="q.name"
                            auto-save
                            v-slot="scope"
                            buttons
                        >
                            <q-input
                                size="30"
                                v-model="scope.value"
                                dense
                                autofocus
                                @keyup.enter="scope.set"
                            />
                        </q-popup-edit>
                    </td>
                    <td>
                        {{ q.description }}
                        <q-popup-edit
                            v-model="q.description"
                            auto-save
                            v-slot="scope"
                            buttons
                        >
                            <q-input
                                size="30"
                                v-model="scope.value"
                                dense
                                autofocus
                                @keyup.enter="scope.set"
                            />
                        </q-popup-edit>
                    </td>
                    <td>
                        <q-btn
                            v-for="b in spec.questions"
                            :key="b.name"
                            :label="b.name"
                            size="xs"
                            @click="editMatch(q, b)"
                        />
                    </td>
                    <td>
                        <q-btn
                            v-for="b in spec.fields"
                            :key="b.name"
                            :label="b.name"
                            size="xs"
                            @click="editField(q, b)"
                        />
                    </td>
                    <td>
                        <q-btn
                            round
                            color="primary"
                            icon="delete"
                            @click="() => spec.services.splice(i, 1)"
                        />
                    </td>
                </tr>
            </tbody>
        </q-markup-table>
        <q-btn
            color="primary"
            @click="() => spec.services.push({})"
            label="Add Service"
        />

        <h4>YAML</h4>
        <q-btn
            color="primary"
            @click="showYAML = !showYAML"
            label="Show YAML"
        />
        <pre v-if="showYAML">{{ yaml.dump(spec) }}</pre>
    </div>
</template>

<style scoped></style>
