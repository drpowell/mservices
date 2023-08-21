<script setup>
import { ssrListen } from "quasar/wrappers";
import { ref, computed, watch } from "vue";

const props = defineProps({
    services: {
        type: Array,
        default: () => [],
    },
    qSelected: Object,
});
const emit = defineEmits(["update:modelValue", "highlight"]);

const sSelected = ref([]);
watch(
    sSelected,
    () => {
        // console.log("emitting", sSelected.value);
        emit("update:modelValue", sSelected.value);
    },
    { deep: true }
);

const sAvailable = computed(() => {
    let res = [];
    for (let i = 0; i < props.services.length; i += 1) {
        if (matches(props.services[i])) {
            res.push(i);
        }
    }
    return res;
});

const toggle = (idx) => {
    if (sAvailable.value.includes(idx)) {
        const i = sSelected.value.indexOf(idx);
        if (i >= 0) {
            sSelected.value.splice(i, 1);
        } else {
            sSelected.value.push(idx);
        }
    }
};
const selAll = (v) => {
    sSelected.value = [];
    if (v) {
        for (let i = 0; i < props.services.length; i += 1) {
            if (sAvailable.value.includes(i)) sSelected.value.push(i);
        }
    }
};

const matches = (service) => {
    for (const [key, options] of Object.entries(props.qSelected)) {
        if (options.length > 0) {
            const match = service.match?.[key] || [];
            if (options.every((o) => match.includes(o))) {
                // Ok, we have this one
            } else {
                return false;
            }
        }
    }
    return true;
};

const enabled = (sIdx) => {
    if (sAvailable.value.includes(sIdx)) return "";
    else return "disable";
};

// Unselect items that are not available based on question changes
watch(sAvailable, () => {
    sSelected.value = sSelected.value.filter((x) =>
        sAvailable.value.includes(x)
    );
});
</script>

<template>
    <div>
        Select the services you would like to compare
        <div class="buttons">
            <q-btn
                outline
                size="sm"
                color="primary"
                label="Select All"
                @click="selAll(true)"
            />
            <q-btn
                outline
                size="sm"
                color="primary"
                label="Clear Selection"
                @click="selAll(false)"
            />
        </div>
    </div>
    <div class="services">
        <div
            v-for="(s, idx) in services"
            :key="idx"
            class="panel"
            :class="enabled(idx)"
            @click="toggle(idx)"
            @mouseover="$emit('highlight', s)"
            @mouseleave="$emit('highlight', null)"
        >
            <q-icon
                class="check"
                size="sm"
                :name="sSelected.includes(idx) ? 'o_check_circle' : 'o_circle'"
            />
            <div class="s-hdr">{{ s.name }}</div>
            <p>{{ s.description }}</p>
        </div>
    </div>
</template>

<style scoped>
.buttons {
    float: right;
    display: inline;
}
.services {
    display: flex;
    flex-wrap: wrap;
    margin-top: 10px;
}

.check {
    float: right;
}

.disable {
    opacity: 50%;
}

.panel {
    position: relative;
    box-sizing: border-box;
    width: calc(20% - 0.5em);
    border-width: 2px;
    border-style: solid;
    border-color: rgb(161, 169, 205);
    border-radius: 0.25em;
    background-color: rgb(228, 230, 240);
    /* height: 7em; */
    min-width: 10em;
    margin: 0.25em;
    padding: 1em;
}
.panel:hover {
    background-color: rgb(198, 199, 207);
}
.s-hdr {
    font-size: 18px;
    font-weight: 500;
}
.panel p {
    font-size: 12px;
    font-weight: 700;
}
</style>
