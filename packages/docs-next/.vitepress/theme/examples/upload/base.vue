<script setup>
import { ref } from "vue";

const file = ref(null);
</script>

<template>
    <section>
        <o-field class="file">
            <o-upload v-slot="{ onclick }" v-model="file">
                <o-button tag="a" variant="primary" @click="onclick">
                    <o-icon icon="upload" />
                    <span>Click to upload</span>
                </o-button>
            </o-upload>
            <span v-if="file" class="file-name">
                {{ file.name }}
            </span>
        </o-field>
    </section>
</template>
