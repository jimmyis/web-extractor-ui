<template>
    <div>
        <div>
            <input type="text"
                @keyup.enter="Go"
                v-model="externalPage"
            >
            <button
                @click="Go"
            >
                Go
            </button>
        </div>
        <vue-i-frame
            class="vue-i-frame"
            :src="pageLoader"
            @load="onLoad"
            @document-load="onDocumentLoad">
        </vue-i-frame>
    </div>
</template>

<script>
import VueIFrame from 'vue-friendly-iframe';

export default {
    data() {
        return {
            externalPage: '',
            pageLoader: 'about:blank',
        }
    },
    components: {
        VueIFrame,
    },
    methods: {
        Go() {
            this.pageLoader = this.externalPage;
        },
        onLoad() {

        },
        onDocumentLoad() {

        }
    }
}
</script>

<style lang="scss">
.vue-i-frame {
    width: 100%;
    min-height: 600px;
    height: 100%;
    display: block;

}

iframe {
    width: 100%;
    min-height: 600px;
    height: 100%;
    display: block;

}
</style>
