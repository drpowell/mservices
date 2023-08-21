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

watch(selectedServices, () => {
    let cs = [{ name: "label", label: "", field: "name" }];
    let fields = [];
    let rs = [];
    for (const [i, s] of selectedServices.value.entries()) {
        cs.push({
            name: s.name,
            label: s.name,
            field: "field" + i,
            align: "left",
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
    //     c.push({ name: s.name });
    //     for (const f of s.fields ?? []) {
    //         if (!r[f.label]) r[f.label] = {};
    //     //     r[f.label][s.name] =
    //     // }
    // }
    // console.log("selServices", selectedServices.value);
    // console.log("cols", cs);
    // console.log("rows", rs);
    columns.value = cs;
    rows.value = rs;
});
</script>

<template>
    <q-table
        v-show="rows.length > 0"
        title="Comparison"
        :rows="rows"
        :columns="columns"
        row-key="name"
        separator="vertical"
        hide-pagination
    >
        <template v-slot:body-cell="props">
            <q-td :props="props">
                <span v-html="props.value"></span>
            </q-td>
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
    background-color: #eee;
    font-weight: bold;
}

.q-table tr:nth-child(even) {
    background-color: #f3f3f3;
}
</style>
