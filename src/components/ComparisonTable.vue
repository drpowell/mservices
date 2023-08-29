<script setup>
import { computed, ref, watch } from "vue";
const props = defineProps({
    services: {
        type: Array,
        default: () => [],
    },
    sSelected: {
        type: Array,
        default: () => [],
    },
});
const selectedServices = computed(() =>
    props.sSelected.map((i) => props.services[i])
);
const columns = ref([]);
const rows = ref([]);
const visibleSel = ref([]);
watch(selectedServices, () => {
    let cs = [{ name: "label", label: "", field: "name", required: true }];
    let fields = [];
    let rs = [];
    visibleSel.value = [];
    for (const [i, s] of selectedServices.value.entries()) {
        visibleSel.value.push(s.name);
        cs.push({
            name: s.name,
            label: s.name,
            field: "field" + i,
            align: "left",
            classes: "table-col",
        });
        for (const f of s.fields ?? []) {
            if (!fields.includes(f.label)) fields.push(f.label);
        }
    }
    for (const f of fields) {
        let r = { name: f };
        for (const [i, s] of selectedServices.value.entries()) {
            r["field" + i] =
                s.fields?.filter((x) => x.label == f)?.[0]?.value ?? "";
        }
        rs.push(r);
    }
    // console.log("selServices", selectedServices.value);
    // console.log("cols", cs);
    // console.log("rows", rs);
    // console.log("visible", visibleSel.value);
    columns.value = cs;
    rows.value = rs;
});

const isVisibleSel = (sel) => {
    return visibleSel.value.includes(sel.name);
};
const toggleSel = (sel) => {
    if (isVisibleSel(sel)) {
        visibleSel.value = visibleSel.value.filter((x) => x != sel.name);
    } else {
        visibleSel.value.push(sel.name);
    }
    // console.log("visible", visibleSel.value);
};
</script>

<template>
    <q-btn
        v-for="s of selectedServices"
        :key="s.name"
        color="primary"
        size="xs"
        rounded
        :outline="!isVisibleSel(s)"
        @click="toggleSel(s)"
    >
        {{ s.name }}
    </q-btn>
    <q-table
        title="Comparison Table"
        :rows="rows"
        :columns="columns"
        :visible-columns="visibleSel"
        no-data-label="No Services Selected"
        row-key="name"
        separator="vertical"
        hide-pagination
    >
        <template v-slot:body-cell="props">
            <q-td :props="props">
                <span v-html="props.value"></span>
            </q-td>
        </template>
        <template v-slot:no-data="{ message }">
            <div class="full-width row flex-center text-accent q-gutter-sm">
                <span> {{ message }} </span>
            </div>
        </template>
    </q-table>
</template>

<style>
.q-table {
    width: 100%;
}

.q-table thead tr:first-child th {
    background-color: var(--q-primary);
    color: #fff;
}

.q-table thead tr th {
    position: sticky;
    z-index: 1;
}

.q-table td:first-child {
    /* background-color: #eee; */
    font-weight: bold;
    width: 0px;
}

.q-table tr:nth-child(even) {
    background-color: #f3f3f3;
}

.q-table .table-col {
    max-width: 20em;
    white-space: pre-wrap;
}
</style>
