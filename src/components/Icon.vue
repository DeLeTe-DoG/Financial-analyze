<template>
    <span class="icon" v-html="icon[variant]"></span>
</template>

<script>
import { default as infoSvg } from "../assets/images/icons/info.svg?raw";

export default {
    props: {
        variant: {
            type: String,
            required: true,
        },
    },
    data() {
        return {
            icon: {
                info: infoSvg,
            },
        };
    },
};
</script>

<style lang="scss">
.icon {
    width: 20px;
    height: 20px;
    svg {
        width: 20px;
        height: 20px;
    }
}
</style>
