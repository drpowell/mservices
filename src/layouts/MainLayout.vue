<script setup></script>

<template>
    <q-layout view="lHh Lpr lFf">
        <q-header elevated>
            <q-toolbar>
                <!-- <q-btn flat dense round icon="menu" aria-label="Menu" /> -->
                <q-toolbar-title> Monash Storage Services </q-toolbar-title>
                <q-avatar square style="width: 100px">
                    <img src="monash-logo-mono.svg" />
                </q-avatar>
            </q-toolbar>
        </q-header>

        <q-page-container>
            <router-view />
        </q-page-container>
    </q-layout>
</template>

<style scoped>
img {
    -webkit-filter: invert(100%); /* safari 6.0 - 9.0 */
    filter: invert(100%);
}
</style>
